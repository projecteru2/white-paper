功能
====

* mount-point

    目前 ELB 最成熟的特色应用是 mount-point 功能，该功能可以将同一 host 的流量分发到多个不同的后端。举例来说，在 `app.yaml` 中，绑定如下:
    ```
        elb:
          - "develop apollo.test.rhllor.net/product"
    ```
    所有到达 ELB host 为 `apollo.test.rhllor.net` 的流量，若 uri 形如 `/product/xxx` 则该流量会被分发到 `product` 对应的后端，同时 uri 会变为 `/xxx`。

    **注意！mount-point 功能并没有最大匹配，它采用的是顺序匹配，转发到第一个匹配项里面，所以，如果有后端A绑定了 `apollo.test.rhllor.net/product` 又有后端B绑定 `apollo.test.rhllor.net/product/whatever` 的话， 后端B会悲剧，因为匹配顺序是按照绑定顺序来的，所以本应到后端B的流量都会到后端A。**