#####DistributedCache

1. 加载SequenceFile(MapFile)到DistributedCache

	1.1 加载

		Configuration conf = getConf();
		Job job = new Job(conf);
		DistributedCache.addCacheFile(new Path(maxWiFilePath), job.getConfiguration());

	1.2 读取
	
	      URI[] localFiles = new URI[0];
	        try {
	            localFiles = DistributedCache.getCacheFiles(conf);//注意不是getLocalCacheFiles()
	        } catch (IOException e) {
	            e.printStackTrace();
	        }

        MapFile.Reader reader = null;

        if (localFiles != null && localFiles.length > 0 && localFiles[0] != null) {
            String mapFileDir = localFiles[0].toString();//如果有多个则要正则匹配;另外注意有的文件名如part-r-00000重名情况
            LOG.info("mapFileDir file:" + mapFileDir);
            FileSystem fs = null;
            try {
                fs = FileSystem.getLocal(conf);
            } catch (IOException e) {
                e.printStackTrace();
            }

            try {
                reader = new MapFile.Reader(new Path(mapFileDir), conf);
            } catch (IOException e) {
                e.printStackTrace();
            }
            if (reader != null) {
                LOG.info("success load cache:" + mapFileDir);

            }


        }
        return reader;

2. 加载其他小文件到DistributedCache

	2.1 加载：与1中相同
	
	2.2 读取：小文件可以直接从本地文件系统读取,这时候可以考虑使用symlink

		BufferedReader br = null;
		br = new BufferedReader(new InputStreamReader(new FileInputStream("DIYFileName")));


