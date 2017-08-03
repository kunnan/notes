**Toast使用详解**

##1.简介

系统提示消息栏


##2.普通用法

	Toast.makeText(this, "123",Toast.LENGTH_LONG).show();

##2.复用用法

	Toast mToast = new Toast(this);
	mToast.setDuration(0);
	mToast.setText("2345");
	mToast.show();
	mToast.cancel();

这样每次使用mToast对象就指挥复用同一个Toast对象。

##3.自定义Toast

实现一个来电归属地显示浮动框

```Java
	/**
	 * 窗体管理者
	 */
	private WindowManager wm;
	private View view;
	WindowManager.LayoutParams params;
	private SharedPreferences sp;
	long[] mHits = new long[2];

	public void myToast(String address) {

		view = View.inflate(this, R.layout.address_show, null);
		TextView tv_address = (TextView) view
				.findViewById(R.id.tv_address);
		
		//双击居中
		view.setOnClickListener(new OnClickListener() {
			
			@Override
			public void onClick(View v) {
				System.arraycopy(mHits, 1, mHits, 0, mHits.length - 1);
				mHits[mHits.length - 1] = SystemClock.uptimeMillis();
				if (mHits[0] >= (SystemClock.uptimeMillis() - 500)) {
					// 双击居中了。。。
					params.x = wm.getDefaultDisplay().getWidth()/2-view.getWidth()/2;
					wm.updateViewLayout(view, params);
					Editor editor = sp.edit();
					editor.putInt("lastx", params.x);
					editor.commit();
				}
				
			}
		});

		view.setOnTouchListener(new OnTouchListener() {

			int startX;
			int startY;

			@Override
			public boolean onTouch(View v, MotionEvent event) {
				// TODO Auto-generated method stub

				switch (event.getAction()) {
				case MotionEvent.ACTION_DOWN: // 手指按下屏幕
					Log.i(TAG, "手指摸到控件");


					startX = (int) event.getRawX();
					startY = (int) event.getRawY();

					break;

				case MotionEvent.ACTION_MOVE: // 手指移动
					
					Log.i(TAG, "手指在控件上移动");


					int newX = (int) event.getRawX();
					int newY = (int) event.getRawY();

					// 变化量
					int dx = newX - startX;
					int dy = newY - startY;

					// 更新ImageView在窗体中的位置
					params.x += dx;
					params.y += dy;

					// 考虑边界
					if (params.x < 0) {
						params.x = 0;
					}
					if (params.y < 0) {
						params.y = 0;
					}
					if (params.x > (wm.getDefaultDisplay().getWidth() - view
							.getWidth())) {
						params.x = (wm.getDefaultDisplay().getWidth() - view
								.getWidth());
					}
					if (params.y > (wm.getDefaultDisplay().getHeight() - view
							.getHeight())) {
						params.y = (wm.getDefaultDisplay().getHeight() - view
								.getHeight());
					}

					wm.updateViewLayout(view, params);

					// 重新初始化手指开始位置
					startX = (int) event.getRawX();
					startY = (int) event.getRawY();
					break;

				case MotionEvent.ACTION_UP: // 手指离开屏幕
					
					Log.i(TAG, "手指离开控件");


					// 记录控件距离屏幕左上角的坐标
					Editor editor = sp.edit();
					editor.putInt("lastX", params.x);
					editor.putInt("lastY", params.y);
					editor.commit();

					break;

				}
		//		return true; // 事件处理完毕,不让父控件或布局响应触摸事件,不能响应点击事件
				return false;//可以响应点击事件
			}
		});

		tv_address.setText(address);

		// "半透明","活力橙","卫士蓝","金属灰","苹果绿"
		int[] ids = { R.drawable.call_locate_white,
				R.drawable.call_locate_orange, R.drawable.call_locate_blue,
				R.drawable.call_locate_gray, R.drawable.call_locate_green };

		sp = getSharedPreferences("config", MODE_PRIVATE);

		view.setBackgroundResource(ids[sp.getInt("which", 0)]);
		tv_address.setText(address);

		// 窗体设置
		params = new WindowManager.LayoutParams();

		params.height = WindowManager.LayoutParams.WRAP_CONTENT;
		params.width = WindowManager.LayoutParams.WRAP_CONTENT;
		// 对话框位置
		params.gravity = Gravity.TOP + Gravity.LEFT;
		// 窗体距左边80 距上边100
		params.x = sp.getInt("lastX", 80);
		params.y = sp.getInt("lastY", 100);

		params.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
				| WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON;
		params.format = PixelFormat.TRANSLUCENT;
		// android系统里面具有电话优先级的一种窗体类型,记得添加权限android.permission.SYSTEM_ALERT_WINDOW
		params.type = WindowManager.LayoutParams.TYPE_PRIORITY_PHONE;
		wm.addView(view, params);

	}
```


