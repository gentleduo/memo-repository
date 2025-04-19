docker run -itd --name=ollama -v /software/deepseek:/root/.ollama -p 1234:11434 ollama/ollama:latest

[root@server01 deepseek]# docker exec -it 9b1bd1dc1a0a /bin/bash

root@9b1bd1dc1a0a:/# ollama pull deepseek-r1:32b