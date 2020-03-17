通过profile启动3个eureka节点

```shell
for i in 1 2 3
        do nohup  java -jar -Dspring.profiles.active=peer${i} eureka.jar > peer${i}.log &
done

```

