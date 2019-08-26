# android-note2-uploadVideo
使用okhttp网络框架来完成对视频的上传，其方法如下：  

public static void uploadVideo(final String url, final String filePath, final Context context,
			final Activity activity) {  
			progressDialog = showRequestDialog(context);  
			progressDialog.show();  
			MultipartBody.Builder builder = new MultipartBody.Builder().setType(MultipartBody.FORM);  
			File file = new File(filePath);  
			if (file.exists()) {  
					RequestBody body = RequestBody.create(MediaType.parse("video/*"), file);  
					builder.addFormDataPart("video", getFileName(), body);  
					builder.addFormDataPart("video", getFileName(), body);  
					MultipartBody postBody = builder.build();  
					Request request = new Request.Builder().url(url).post(postBody).build();  
					OkHttpClient client = new OkHttpClient();  
					Call call = client.newCall(request);    
					
					call.enqueue(new Callback() {  

							@Override
							public void onResponse(Call arg0, Response resp) throws IOException {
					// TODO Auto-generated method stub
							String resp_str = resp.body().string();
							String msg = "";
							try {
								JSONObject object = new JSONObject(resp_str);
								msg = object.getString("result");
									} catch (JSONException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					if ("0".equals(msg)) {
						activity.runOnUiThread(new Runnable() {

							@Override
							public void run() {
								// TODO Auto-generated method stub
								progressDialog.dismiss();
								Toast.makeText(context, "上传成功", Toast.LENGTH_SHORT).show();
								Intent intent=new Intent();
								intent.setClass(activity, MainActivity.class);
								activity.startActivity(intent);
								activity.finish();
							}
						});
					} else {
						activity.runOnUiThread(new Runnable() {

							@Override
							public void run() {
								// TODO Auto-generated method stub
								progressDialog.dismiss();
								Toast.makeText(context, "上传失败", Toast.LENGTH_SHORT).show();	
							}
						});
					}

				}

				@Override
				public void onFailure(Call arg0, IOException arg1) {
					// TODO Auto-generated method stub

				}
			});
		}

	}

	private static String getFileName() {
		SimpleDateFormat format = new SimpleDateFormat("YYYYMMDD_HHmmss");
		String fileName = format.format(new Date());
		fileName = fileName + ".mp4";
		return fileName;
	}

	private static ProgressDialog showRequestDialog(Context context) {
		ProgressDialog dialog = new ProgressDialog(context);
		dialog.setTitle("正在上传数据");
		dialog.setMessage("请稍后....");
		dialog.setCancelable(false);
		return dialog;
	}
