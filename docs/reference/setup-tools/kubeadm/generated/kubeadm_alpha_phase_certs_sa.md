
生成私钥用于与公钥一起签署服务账户令牌

### 概要




生成私钥用于与公钥一起签署服务账户令牌，并把它们存储在 sa.key 和 sa.pub 文件中。

如果这两个文件都存在，kubeadm会跳过生成步骤并使用现存的文件。

Alpha 免责声明：此命令处于 alpha 阶段。

```
kubeadm alpha phase certs sa
```



### 选择项

```
      --cert-dir string   证书存储路径（默认值 "/etc/kubernetes/pki"）
      --config string     kubeadm config 文件路径（警告：配置文件的使用是实验性的）
```
