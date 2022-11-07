=== "临时DNS"

    如果您正在使用`curl`访问样例应用程序，或者您自己的Knative应用程序，并且不能使用"Magic DNS (sslip.io)"或“Real DNS”方法，那么有一种临时的方法。
    这对于那些希望在不更改DNS配置的情况下评估Knative的人是很有用的，就像"Real DNS"方法一样，或者因为使用本地minikube或IPv6集群而不能使用 "Magic DNS"方法。

    要使用curl访问你的应用程序，使用以下方法:

    1. 启动应用程序后，获取应用程序的URL:
      ```bash
      kubectl get ksvc
      ```
      The output should be similar to:
      ```bash
      NAME            URL                                        LATESTCREATED         LATESTREADY           READY   REASON
      helloworld-go   http://helloworld-go.default.example.com   helloworld-go-vqjlf   helloworld-go-vqjlf   True
      ```

    1. Instruct `curl` to connect to the External IP or CNAME defined by the
       networking layer mentioned in section 3, and use the `-H "Host:"` command-line
       option to specify the Knative application's host name.
       For example, if the networking layer defines your External IP and port to be `http://192.168.39.228:32198` and you wish to access the `helloworld-go` application mentioned earlier, use:
       ```bash
       curl -H "Host: helloworld-go.default.example.com" http://192.168.39.228:32198
       ```
       In the case of the provided `helloworld-go` sample application, using the default configuration, the output is:
       ```
       Hello Go Sample v1!
       ```
       Refer to the "Real DNS" method for a permanent solution.
