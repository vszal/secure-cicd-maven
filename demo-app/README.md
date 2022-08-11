# demo-app 

## Create a Spring Boot Application

Get the source code
```shell
git clone git@github.com:ddobrin/modern-java-google-cloud.git
```

Validate that the download is correct
```shell
cd demo-app

./mvnw clean spring-boot:run

# using cUrl
curl localhost:8080

Output: 
Hello from your local environment!

# using HTTPie
http GET :8080/hello
HTTP/1.1 200
...
Hello from your local environment!

#stop the app with CTRL-C
```

### Note: The app is ready to be deployed to GCP!

