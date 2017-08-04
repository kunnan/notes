
**Android启动流程①init进程分析**

[TOC]

# 简介

当Linux内核启动之后，运行的第一个进程是init，这个进程是一个守护进程，它的生命周期贯穿整个linux 内核运行的始终， linux中所有其它的进程的共同始祖均为init进程。 
Android系统是运作在linux内核上的，为了启动并运行整个android系统，google实现了android系统的init进程。s

# init主函数

>Android init进程的入口文件在system/core/init/init.cpp中。

```c
int main(int argc, char** argv) {

    //  创建设备节点文件
    if (!strcmp(basename(argv[0]), "ueventd")) {
        return ueventd_main(argc, argv);
    }
    
    //Linux看门狗，用于监测系统的运行
    if (!strcmp(basename(argv[0]), "watchdogd")) {
        return watchdogd_main(argc, argv);
    }

    // 清除屏蔽字，保证新建的目录的访问权限不受屏蔽字的影响
    umask(0);

    add_environment("PATH", _PATH_DEFPATH);

    bool is_first_stage = (argc == 1) || (strcmp(argv[1], "--second-stage") != 0);

    // 创建和挂载启动所需的文件目录
    if (is_first_stage) {
        mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
        mkdir("/dev/pts", 0755);
        mkdir("/dev/socket", 0755);
        mount("devpts", "/dev/pts", "devpts", 0, NULL);
        #define MAKE_STR(x) __STRING(x)
        mount("proc", "/proc", "proc", 0, "hidepid=2,gid=" MAKE_STR(AID_READPROC));
        mount("sysfs", "/sys", "sysfs", 0, NULL);
    }

    // 屏蔽标准的输入输出
    open_devnull_stdio();
    
    //初始化内核log系统
    klog_init();
    klog_set_level(KLOG_NOTICE_LEVEL);

    NOTICE("init %s started!\n", is_first_stage ? "first stage" : "second stage");

    if (!is_first_stage) {
        // Indicate that booting is in progress to background fw loaders, etc.
        close(open("/dev/.booting", O_WRONLY | O_CREAT | O_CLOEXEC, 0000));
        //初始化属性相关资源
        property_init();

        ···
    }

    // Set up SELinux
    selinux_initialize(is_first_stage);

    ···
    
    //创建epoll句柄
    epoll_fd = epoll_create1(EPOLL_CLOEXEC);
    
    ···

    signal_handler_init();

    property_load_boot_defaults();
    export_oem_lock_status();
    
    //启动属性相关服务
    start_property_service();

    const BuiltinFunctionMap function_map;
    Action::set_function_map(&function_map);

    Parser& parser = Parser::GetInstance();
    parser.AddSectionParser("service",std::make_unique<ServiceParser>());
    parser.AddSectionParser("on", std::make_unique<ActionParser>());
    parser.AddSectionParser("import", std::make_unique<ImportParser>());
    
    //解析init.c配置文件
    parser.ParseConfig("/init.rc");

    //向执行队列中添加其它action 
    ActionManager& am = ActionManager::GetInstance();

    ···
    ···
    ···

    while (true) {
        //判断是否有事件需要处理
        if (!waiting_for_exec) {
            //依次执行每个action中携带command对应的执行函数
            am.ExecuteOneCommand();
            //重启一些挂掉的进程
            restart_processes();
        }

        int timeout = -1;
        ////有进程需要重启时，等待该进程重启
        if (process_needs_restart) {
            timeout = (process_needs_restart - gettime()) * 1000;
            if (timeout < 0)
                timeout = 0;
        }

        //有action待处理，不等待
        if (am.HasMoreCommands()) {
            timeout = 0;
        }

        bootchart_sample(&timeout);

        epoll_event ev;
        //没有事件到来的话，最多阻塞timeout时间
        int nr = TEMP_FAILURE_RETRY(epoll_wait(epoll_fd, &ev, 1, timeout));
        if (nr == -1) {
            ERROR("epoll_wait failed: %s\n", strerror(errno));
        } else if (nr == 1) {
            //有事件到来，执行对应处理函数
            //根据上文知道，epoll句柄（即epoll_fd）主要监听子进程结束，及其它进程设置系统属性的请求。
            ((void (*)()) ev.data.ptr)();
        }
    }

    return 0;
}
```

init主函数完成了许多功能，主要实现的功能有如下：

* `property_init()`初始化属性服务
* `start_property_service()`启动属性服务
* `parser.ParseConfig("/init.rc")`解析init.rc配置文件
* 将所有需要操作的action加入到ActionManager管理的队列中，进入无线循环过程，不断处理运行队列中的事件，同时进行重启service等操作。

接下来了解属性服务：

## 属性服务

>system/core/init/init.cpp

```
property_init();
start_property_service()
```
### property_init()

>属性初始化，init进程在共享内存区域中，创建并初始化属性域。
>property_init()函数的具体实现在system/core/init/property_service.cpp 

```cpp
void property_init() {
    //初始化属性内存区域
    if (__system_property_area_init()) {
        ERROR("Failed to initialize property area\n");
        exit(1);
    }
}
```

### start_property_service()

>启动配置属性服务端
>system/core/init/property_service.cpp

```cpp
void start_property_service() {
    //创建一个非阻塞的socket
    property_set_fd = create_socket(PROP_SERVICE_NAME, SOCK_STREAM | SOCK_CLOEXEC | SOCK_NONBLOCK,
                                    0666, 0, 0, NULL);
    if (property_set_fd == -1) {
        ERROR("start_property_service socket creation failed: %s\n", strerror(errno));
        exit(1);
    }
    //监听property_set_fd，使得该socket变成一个服务
    listen(property_set_fd, 8);
    //将property_set_fd放入epoll句柄中，用epoll来监听property_set_fd，当有数据到来时，init进程将使用handle_property_set_fd函数进行处理
    register_epoll_handler(property_set_fd, handle_property_set_fd);
}
```

调用listen后，init进程成为一个服务进程，其它进程可以通过property_set_fd连接init进程，提交设置系统属性的申请。在启动配置属性服务的最后，调用了register_epoll_handler函数，该函数将利用之前创建出的epoll句柄监听property_set_fd。当property_set_fd有数据到来时，init进程将使用handle_property_set_fd()函数进行处理

```cpp
void register_epoll_handler(int fd, void (*fn)()) {
    epoll_event ev;
    ev.events = EPOLLIN;
    ev.data.ptr = reinterpret_cast<void*>(fn);
    //epoll_fd增加一个监听对象fd,fd上有数据到来时，调用fn处理
    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, fd, &ev) == -1) {
        ERROR("epoll_ctl failed: %s\n", strerror(errno));
    }
}

//数据处理函数
static void handle_property_set_fd()
{
    
    ···

    //监听连接请求
    if ((s = accept(property_set_fd, (struct sockaddr *) &addr, &addr_size)) < 0) {
        return;
    }

   ···
    //  接受数据
    r = TEMP_FAILURE_RETRY(recv(s, &msg, sizeof(msg), MSG_DONTWAIT));
    // 处理数据
    switch(msg.cmd) {
    case PROP_MSG_SETPROP:
        
        ···

        if(memcmp(msg.name,"ctl.",4) == 0) {
            
            close(s);
            if (check_control_mac_perms(msg.value, source_ctx, &cr)) {
                handle_control_message((char*) msg.name + 4, (char*) msg.value);
            } else {
                ERROR("sys_prop: Unable to %s service ctl [%s] uid:%d gid:%d pid:%d\n",
                        msg.name + 4, msg.value, cr.uid, cr.gid, cr.pid);
            }
        } else {
            //检查进程权限
            if (check_mac_perms(msg.name, source_ctx, &cr)) {
                //修改属性
                property_set((char*) msg.name, (char*) msg.value);
            } else {
                ERROR("sys_prop: permission denied uid:%d  name:%s\n",
                      cr.uid, msg.name);
            }
            close(s);
        }
        freecon(source_ctx);
        break;

    default:
        close(s);
        break;
    }
}
```

handle_property_set_fd（）函数实际上调用accept函数监听连接请求，recv接收到来的数据，然后根据数据类型，进行设置系统属性等相关操作。

>init进程在共享内存区域中，创建并初始化属性域。其它进程可以访问属性域中的值，但更改属性值仅能在init进程中进行。这就是init进程调用start_property_service的原因。其它进程修改属性值时，要预先向init进程提交值变更申请，然后init进程处理该申请，并修改属性值。在访问和修改属性时，init进程都可以进行权限控制。所以在修改属性时，通过check_mac_perms()函数检查进程权限。

属性修改则调用property_set（）实现：

```cpp
//属性修改
int property_set(const char* name, const char* value) {
    int rc = property_set_impl(name, value);
    if (rc == -1) {
        ERROR("property_set(\"%s\", \"%s\") failed\n", name, value);
    }
    return rc;
}
```
property_set函数则通过调用property_set_impl函数来实现：

```cpp
static int property_set_impl(const char* name, const char* value) {
    size_t namelen = strlen(name);
    size_t valuelen = strlen(value);

    if (!is_legal_property_name(name, namelen)) return -1;
    if (valuelen >= PROP_VALUE_MAX) return -1;

    if (strcmp("selinux.reload_policy", name) == 0 && strcmp("1", value) == 0) {
        if (selinux_reload_policy() != 0) {
            ERROR("Failed to reload policy\n");
        }
    } else if (strcmp("selinux.restorecon_recursive", name) == 0 && valuelen > 0) {
        if (restorecon_recursive(value) != 0) {
            ERROR("Failed to restorecon_recursive %s\n", value);
        }
    }

    prop_info* pi = (prop_info*) __system_property_find(name);

    if(pi != 0) {
        /* ro.* 表示只读，不能修改，直接返回 */
        if(!strncmp(name, "ro.", 3)) return -1;

        __system_property_update(pi, value, valuelen);
    } else {
        //属性不存在则添加该属性
        int rc = __system_property_add(name, namelen, value, valuelen);
        if (rc < 0) {
            return rc;
        }
    }
    /* If name starts with "net." treat as a DNS property. */
    if (strncmp("net.", name, strlen("net.")) == 0)  {
        if (strcmp("net.change", name) == 0) {
            return 0;
        }
       /*
        * 以net.开头的属性名称更新后，需要将属性名称写入net.change中
        */
        property_set("net.change", name);
    } else if (persistent_properties_loaded &&
            strncmp("persist.", name, strlen("persist.")) == 0) {
        /*
         * Don't write properties to disk until after we have read all default properties
         * to prevent them from being overwritten by default values.
         */
        write_persistent_property(name, value);
    }
    property_changed(name, value);
    return 0;
}
```

属性服务的工作过程基本就是这样，下面来看看init.rc文件的解析

## init.rc

init.rc文件是在init进程启动后执行的启动脚本，文件中记录着init进程需执行的操作。在Android系统中，使用init.rc和init.{ hardware }.rc两个文件。

```cpp
Parser& parser = Parser::GetInstance();
parser.AddSectionParser("service",std::make_unique<ServiceParser>());
parser.AddSectionParser("on", std::make_unique<ActionParser>());
parser.AddSectionParser("import", std::make_unique<ImportParser>());
    
//解析init.c配置文件
parser.ParseConfig("/init.rc");
```

此处解析函数传入的参数为“/init.rc”，解析的是运行时与init进程同在根目录下的init.rc文件。该文件在编译前，定义于system/core/rootdir/init.rc中,可以看到

```cpp
import /init.${ro.zygote}.rc
```
从import的文件命名方式，我们可以看出在不同的平台（32、64及64_32）上，init.rc将包含不同的zygote.rc文件。不同的zygote.rc内容大致相同，主要区别体现在启动的是32位，还是64位的进程。这里拿64位处理器为例，init.zygote64.rc的代码如下所示。
>system/core/rootdir/init.zygote64.rc

```cpp
service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server
    class main //指的是zygote的class name为main
    socket zygote stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    writepid /dev/cpuset/foreground/tasks
```

>可以看到init进程将会zygote进程，这个进程的执行路径为`/system/bin/app_process64`，后面则是要传给app_process64的参数。

接下来看看系统如何解析service。

## service解析

```cpp
parser.ParseConfig("/init.rc");

bool Parser::ParseConfig(const std::string& path) {
    if (is_dir(path.c_str())) {
        //实际上调用ParseConfigFile(path)
        return ParseConfigDir(path);
    }
    return ParseConfigFile(path);
}

bool Parser::ParseConfigFile(const std::string& path) {
    
    ···
    
    ParseData(path, data);
    
    ···
    
    return true;
}

void Parser::ParseData(const std::string& filename, const std::string& data) {
    
    ···

    parse_state state;
  
    ···

    SectionParser* section_parser = nullptr;
    std::vector<std::string> args;

    for (;;) {
        //next_token以行为单位分割参数传递过来的字符串
        //最先走到T_TEXT分支
        switch (next_token(&state)) {
        case T_EOF:
            if (section_parser) {
                //EOF,解析结束
                section_parser->EndSection();
            }
            return;
        case T_NEWLINE:
            state.line++;
            if (args.empty()) {
                break;
            }
            //在前面创建parser时，我们为service，on，import定义了对应的parser 
            //这里就是根据第一个参数，判断是否有对应的parser
            if (section_parsers_.count(args[0])) {
                if (section_parser) {
                    section_parser->EndSection();
                }
                //获取参数对应的parser
                section_parser = section_parsers_[args[0]].get();
                std::string ret_err;
                //调用实际parser的ParseSection函数
                if (!section_parser->ParseSection(args, &ret_err)) {
                    parse_error(&state, "%s\n", ret_err.c_str());
                    section_parser = nullptr;
                }
            } else if (section_parser) {
                std::string ret_err;
                //如果第一个参数不是service，on，import
                //则调用前一个parser的ParseLineSection函数
                //这里相当于解析一个参数块的子项
                if (!section_parser->ParseLineSection(args, state.filename,
                                                      state.line, &ret_err)) {
                    parse_error(&state, "%s\n", ret_err.c_str());
                }
            }
            //清空本次解析的数据
            args.clear();
            break;
        case T_TEXT:
        //将本次解析的内容写入到args中
            args.emplace_back(state.text);
            break;
        }
    }
}

```
从上述代码可知，service的rc文件最终解析是由ServiceParser的函数解析的，比如init.zygote64.rc，接下来看看ServiceParser的函数：函数位于`system/core/init/service.cpp`

```cpp
bool ServiceParser::ParseSection(const std::vector<std::string>& args,
                                 std::string* err) {
    if (args.size() < 3) {
        *err = "services must have a name and a program";
        return false;
    }

    const std::string& name = args[1];
    if (!IsValidName(name)) {
        *err = StringPrintf("invalid service name '%s'", name.c_str());
        return false;
    }

    std::vector<std::string> str_args(args.begin() + 2, args.end());
    //构造出一个service对象，它的classname为”default”
    service_ = std::make_unique<Service>(name, "default", str_args);
    return true;
}

//解析子项
bool ServiceParser::ParseLineSection(const std::vector<std::string>& args,
                                     const std::string& filename, int line,
                                     std::string* err) const {
    return service_ ? service_->HandleLine(args, err) : false;
}

//解析完毕时调用
void ServiceParser::EndSection() {
    if (service_) {
        //将service对象加入到services链表中
        ServiceManager::GetInstance().AddService(std::move(service_));
    }
}

void ServiceManager::AddService(std::unique_ptr<Service> service) {
    Service* old_service = FindServiceByName(service->name());
    if (old_service) {
        ERROR("ignored duplicate definition of service '%s'",
              service->name().c_str());
        return;
    }
    services_.emplace_back(std::move(service));
}
```

## init启动zygote

接下来看看init是如何启动service，在这里主要了解启动zygote这个service。在zygote的启动脚本中我们得知zygote的class name为main。在init.rc有如下配置代码：
system/core/rootdir/init.rc

```
on nonencrypted
    # A/B update verifier that marks a successful boot.
    exec - root cache -- /system/bin/update_verifier nonencrypted
    class_start main //main指的就是zygote
    class_start late_start
```
那么系统是是如何处理上面的脚本的呢？在init.cpp的main函数中有

```
const BuiltinFunctionMap function_map;
Action::set_function_map(&function_map);
```

因此，Action中调用function_map_->FindFunction时，实际上调用的是BuiltinFunctionMap的FindFunction函数。我们已经知道FindFunction是keyword定义的通用函数，重点是重构的map函数。我们看看system/core/init/builtins.cpp：

```cpp
BuiltinFunctionMap::Map& BuiltinFunctionMap::map() const {
    constexpr std::size_t kMax = std::numeric_limits<std::size_t>::max();
    static const Map builtin_functions = {
        {"bootchart_init",          {0,     0,    do_bootchart_init}},
        {"chmod",                   {2,     2,    do_chmod}},
        {"chown",                   {2,     3,    do_chown}},
        {"class_reset",             {1,     1,    do_class_reset}},
        {"class_start",             {1,     1,    do_class_start}},
        {"class_stop",              {1,     1,    do_class_stop}},
        {"copy",                    {2,     2,    do_copy}},
        {"domainname",              {1,     1,    do_domainname}},
        {"enable",                  {1,     1,    do_enable}},
        {"exec",                    {1,     kMax, do_exec}},
        {"export",                  {2,     2,    do_export}},
        {"hostname",                {1,     1,    do_hostname}},
        {"ifup",                    {1,     1,    do_ifup}},
        {"init_user0",              {0,     0,    do_init_user0}},
        {"insmod",                  {1,     kMax, do_insmod}},
        {"installkey",              {1,     1,    do_installkey}},
        {"load_persist_props",      {0,     0,    do_load_persist_props}},
        {"load_system_props",       {0,     0,    do_load_system_props}},
        {"loglevel",                {1,     1,    do_loglevel}},
        {"mkdir",                   {1,     4,    do_mkdir}},
        {"mount_all",               {1,     kMax, do_mount_all}},
        {"mount",                   {3,     kMax, do_mount}},
        {"powerctl",                {1,     1,    do_powerctl}},
        {"restart",                 {1,     1,    do_restart}},
        {"restorecon",              {1,     kMax, do_restorecon}},
        {"restorecon_recursive",    {1,     kMax, do_restorecon_recursive}},
        {"rm",                      {1,     1,    do_rm}},
        {"rmdir",                   {1,     1,    do_rmdir}},
        {"setprop",                 {2,     2,    do_setprop}},
        {"setrlimit",               {3,     3,    do_setrlimit}},
        {"start",                   {1,     1,    do_start}},
        {"stop",                    {1,     1,    do_stop}},
        {"swapon_all",              {1,     1,    do_swapon_all}},
        {"symlink",                 {2,     2,    do_symlink}},
        {"sysclktz",                {1,     1,    do_sysclktz}},
        {"trigger",                 {1,     1,    do_trigger}},
        {"verity_load_state",       {0,     0,    do_verity_load_state}},
        {"verity_update_state",     {0,     0,    do_verity_update_state}},
        {"wait",                    {1,     2,    do_wait}},
        {"write",                   {2,     2,    do_write}},
    };
    return builtin_functions;
}
```
第四项就是就是的Action nonencrypted 的命令class_start所对应的执行函数，执行函数为do_class_start，接下来看看这个函数：

system/core/init/builtins.cpp

```cpp
static int do_class_start(const std::vector<std::string>& args) {
        /* Starting a class does not start services
         * which are explicitly disabled.  They must
         * be started individually.
         */
    ServiceManager::GetInstance().
        ForEachServiceInClass(args[1], [] (Service* s) { s->StartIfNotDisabled(); });
    return 0;
}
```
来查看StartIfNotDisabled做了什么：
system/core/init/service.cpp

```cpp
bool Service::StartIfNotDisabled() {
    if (!(flags_ & SVC_DISABLED)) {
        return Start();
    } else {
        flags_ |= SVC_DISABLED_START;
    }
    return true;
}
```
继续看Start()方法：

```cpp
bool Service::Start() {

    flags_ &= (~(SVC_DISABLED|SVC_RESTARTING|SVC_RESET|SVC_RESTART|SVC_DISABLED_START));
    time_started_ = 0;

    // Service已经运行，则不启动
    if (flags_ & SVC_RUNNING) {
        return false;
    }

    bool needs_console = (flags_ & SVC_CONSOLE);
    if (needs_console && !have_console) {
        ERROR("service '%s' requires console\n", name_.c_str());
        flags_ |= SVC_DISABLED;
        return false;
    }
    
    //判断需要启动的Service的对应的执行文件是否存在，不存在则不启动该Service
    struct stat sb;
    if (stat(args_[0].c_str(), &sb) == -1) {
        ERROR("cannot find '%s' (%s), disabling '%s'\n",
              args_[0].c_str(), strerror(errno), name_.c_str());
        flags_ |= SVC_DISABLED;
        return false;
    }

    ···
    ···
   
    NOTICE("Starting service '%s'...\n", name_.c_str());
    
    //1.fork子进程
    pid_t pid = fork();
    //成功创建子进程
    if (pid == 0) {
        umask(077);

        for (const auto& ei : envvars_) {
            add_environment(ei.name.c_str(), ei.value.c_str());
        }

        for (const auto& si : sockets_) {
            int socket_type = ((si.type == "stream" ? SOCK_STREAM :
                                (si.type == "dgram" ? SOCK_DGRAM :
                                 SOCK_SEQPACKET)));
            const char* socketcon =
                !si.socketcon.empty() ? si.socketcon.c_str() : scon.c_str();

            int s = create_socket(si.name.c_str(), socket_type, si.perm,
                                  si.uid, si.gid, socketcon);
            if (s >= 0) {
                PublishSocket(si.name, s);
            }
        }

        ···
        ···
        
        //2 通过execve执行程序
        if (execve(strs[0], (char**) &strs[0], (char**) ENV) < 0) {
            ERROR("cannot execve('%s'): %s\n", strs[0], strerror(errno));
        }

        _exit(127);
    }

    ···

    NotifyStateChange("running");
    return true;
}
```
Start方法中调用fork函数来创建子进程，并在子进程中调用execve执行system/bin/app_process64，这样就会进入framework/cmds/app_process/app_main.cpp的main函数，如下所示。
frameworks/base/cmds/app_process/app_main.cpp

```cpp
int main(int argc, char* const argv[])
{
    ...
    if (zygote) {
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);//1
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
        return 10;
    }
}
```
系统最终调用runtime(AppRuntime)的start来启动zygote。

# 小结

init进程的主要功能：

* 创建和挂载设备
* 初始化和启动属性服务
* 解析init.rc文件并启动zygote进程

下面将要分析zygote进程的启动过程。

# 参考资料

* [Android6.0 ueventd](http://blog.csdn.net/gaugamela/article/details/52277126)
* [Android6.0 watchdog](http://blog.csdn.net/gaugamela/article/details/52274067)
* [Android系统启动流程（一）解析init进程启动过程](http://liuwangshu.cn/framework/booting/1-init.html)
* [Android7.0 init进程源码分析](http://blog.csdn.net/gaugamela/article/details/52133186)

