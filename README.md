# Building-OpenGFW-on-OpenWrt

## setup.1
Prepare a device capable of running OpenWrt. Here, for demonstration purposes, we'll use Proxmox Virtual Environment (PVE) for deployment. Please search online for installation instructions.

- Architecture: amd64
- Firmware: iStore OS
- Demo: Windows 10

## setup.2
Once the system is installed, please SSH into it and then open a browser to access the control panel `192.168.100.1`. The username is `root` and the password is `password`.

- Update the package repositories.
> Location: System > Software
> ![image](https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_186e9a9c6465f262b0b87fd7d2e8a4d2.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1708777595&Signature=%2FqcPPhurKN6Iqeg2FaAJmQZ4NDg%3D)

- Update the package repositories.
> ![image](https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_0b8bbe57073e566298ad57914118366f.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1708771262&Signature=GWlj4D9iktXOKN7Lzq7PRFqrIt8%3D)

- Install `golang` and its extensions.
> Due to the outdated version 1.19 provided by OpenWrt, compilation is not feasible. You'll need to use the latest version 1.22, which can be obtained from [here](https://github.com/ParrotXray/Building-OpenGFW-on-OpenWrt/releases/tag/v1.22.0). Please download according to your architecture.
> - golang_1.22.0-1_x86_64.ipk
> - golang-src_1.22.0-1_x86_64.ipk
> - golang-doc_1.22.0-1_x86_64.ipk
> 
> ![image](https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_fd51a21b90bb9f71409ea4d50e6b2a80.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1708771289&Signature=LFh6tAu4OWzStQ2oZIaqycZLiPM%3D)

- Install Git.
> Install the required packages as shown in the image below.
> ![image](https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_6f47fd20fb3d13f04dbc287ad5e906a9.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1708771337&Signature=JFDszozRDyJ34ge8pyK2pgOekw4%3D)

- Open an SSH session and clone the [OpenGFW source code](https://github.com/apernet/OpenGFW.git).
```sh=
git clone https://github.com/apernet/OpenGFW.git
cd OpenGFW
```
- Install dependencies
```sh=
opkg install kmod-nft-queue kmod-nf-conntrack-netlink iptables-mod-nfqueue
```
- Begin building the source code.
```sh=
go build
```
- Create configuration files and rule files.
```sh=
vim config.yaml
```
```sh=
# config
io:
  queueSize: 1024
  local: false # Note that the 'router' option should be set to 'false' to avoid blocking issues.

workers:
  count: 4
  queueSize: 16
  tcpMaxBufferedPagesTotal: 4096
  tcpMaxBufferedPagesPerConn: 64
  udpMaxStreams: 4096
```
```sh=
vim rules.yaml
```
```sh=
# rules
# block bilibili
- name: block bilibili http
  action: block
  expr: string(http?.req?.headers?.host) endsWith "bilibili.com"

- name: block bilibili https
  action: block
  expr: string(tls?.req?.sni) endsWith "bilibili.com"

# block csdn
- name: block csdn http
  action: block
  expr: string(http?.req?.headers?.host) endsWith "csdn.net"

- name: block csdn https
  action: block
  expr: string(tls?.req?.sni) endsWith "csdn.net"
  
# block github
- name: block github http
  action: block
  expr: string(http?.req?.headers?.host) endsWith "github.com"

- name: block github https
  action: block
  expr: string(tls?.req?.sni) endsWith "github.com"
```

- Start the process.
```sh=
export OPENGFW_LOG_LEVEL=debug
./OpenGFW -c config.yaml rules.yaml
```

## setup.3
Let's demonstrate the results by creating a virtual machine running Windows 10 to observe the effects.

- Please ensure that OpenWrt has successfully assigned an IP address to the Windows 10 virtual machine.
![image](https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_df75ba4ac9fdfb2589b189dc38b38187.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1708771355&Signature=y9SylPuHiaHCDIRohiKVNkXqUSg%3D)

- Open a browser and attempt to access the URL specified in the `relus.yaml` file. If you are unable to access it, then the setup is successful.
![image](https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_0eaf006f8804bdd8fb543b8c9e72a608.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1708771369&Signature=aWtDaRivA%2BzkkbYZVAiwdxZfDME%3D)

- The blocking records in the log file.
![image](https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_d0eacedb3821640bab2bb9e7932bf82c.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1708771384&Signature=E87b9q8ALukWbfp77nLnUI%2BXw14%3D)

## END
**Reference materials**
- OpenGFW github → [link](https://github.com/apernet/OpenGFW)
- PVE + OpenWrt → [link](https://api.wolfx.jp/websocket.html)
