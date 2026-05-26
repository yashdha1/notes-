using to simulate the aws cloud runnning inside of a docker container. 
```shell
	localstack start -d
``` 
use the commands inside of the exec of the container only. 
also make sure you have the awslocal installed to actually run the commands. 

```shell
awslocal --version
```

1. list all the s3 buckets running. 
```shell 
awslocal s3 ls
```

2. creating a s3 buckets
```shell 
	awslocal s3 mb s3://bukcket_name
```




































