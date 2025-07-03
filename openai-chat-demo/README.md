
### Start the app

Before starting the application make sure you set the `OPENAI_API_KEY` environment variable

### Example 

```bash
http :8080/api/chat\?prompt="tell me a joke"
HTTP/1.1 200
Connection: keep-alive
Content-Length: 77
Content-Type: text/plain;charset=UTF-8
Date: Thu, 03 Jul 2025 14:50:09 GMT
Keep-Alive: timeout=60

Why did the scarecrow win an award?

Because he was outstanding in his field!
```

### Native build

```bash
mvn -Pnative spring-boot:build-image

# docker image created:
docker image ls | grep openai-chat-demo
openai-chat-demo                           0.0.1-SNAPSHOT   bffb0ccfba2a   45 years ago   287MB

# set OPENAI_API_KEY
export OPENAI_API_KEY=

# start 
docker run -e OPENAI_API_KEY=$OPENAI_API_KEY --rm -p 8080:8080 openai-chat-demo:0.0.1-SNAPSHOT

# test
http :8080/api/chat\?prompt="Hello"
Hello! How can I assist you today?
```




### Resources:

- https://www.youtube.com/watch?v=pBVKkcBhw6I