# 后台接收上传的数据的方法  
//保存上传视频的方法  
    @RequestMapping(value="saveVideo.html")  
    public void uploadVideo(MultipartHttpServletRequest request,HttpServletResponse response){  
        System.out.println("saveVideo.html--------我收到了通知");  
        String msg="-1";  
        Iterator<String> iter=request.getFileNames();  
        MultipartFile file=request.getFile("video");  
        String fileName=file.getOriginalFilename();  
        String path = "/upload/video/";  
        //文件保存的路径  
        String rootName =  request.getSession().getServletContext().getRealPath(path);  
        String fileSavePath = rootName;  
        //上传文件目录  
        File uploadPath = new File(fileSavePath);  
        if(!uploadPath.exists()){  
            uploadPath.mkdirs();  
        }  
        File uploadFile=new File(rootName,fileName);  
        try {  
            //这里使用Apache的FileUtils方法来进行保存  
            FileUtils.copyInputStreamToFile(file.getInputStream(), uploadFile);  
            msg="0";  
        } catch (IOException e) {  
            msg="-1";  
            System.out.println("文件上传失败"+e.getMessage());  
        }  
        //将结果返回给客户端  
        JSONObject object=new JSONObject();  
        object.put("result",msg);  
        PrintWriterUtil.print(object.toString(),response);  
    }  
