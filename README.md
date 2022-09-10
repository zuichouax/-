

# -# -*- coding: utf-8 -*-
import os
import random
import re
from time import sleep
from wmi import WMI

from time import sleep
from subprocess import run, PIPE

# 随机修改指定ip段的本机ip
class updateIP:
    def __init__(self):
        self.wmiService = WMI()
        # 获取到本地有网卡信息
        self.colNicConfigs = self.wmiService.Win32_NetworkAdapterConfiguration(
            IPEnabled=True)

    def getAdapter(self):
        flag = 0
        # 遍历所有网卡，找到要修改的那个，这里我是用原ip的第一段正则出来的
        for obj in self.colNicConfigs:
            ip = re.findall("10.\d+.\d+.\d+", obj.IPAddress[0])
            # ip = re.findall("192.\d+.\d+.\d+", obj.IPAddress[0])
            if len(ip) > 0:
                # print(self.colNicConfigs[flag])
                # print("网卡为",flag+1)
                print("\033[33m")
                print("现在使用ip为:",ip)
                print("\033[0m")
                return flag
            else:
                flag = flag + 1


    def runSet(self,num):
        getSubnetMasks = []
        getDefaultGateways = []
        getDNSServers = []

        adapter = self.colNicConfigs[self.getAdapter()]

        if num == 1:
            print("\033[33m自动随机数搜索\033[0m")
        elif num == 0:
            print("\033[33m常规自定义搜索\033[0m")

        # 提取本机IP
        getip = (adapter.IPAddress)
        # 拆分ip字段
        ipv4=getip[0].split(sep = ".") 

        # 提取子网掩码
        getSubnetMasks.append(adapter.IPSubnet[0])
        print("现在子网掩码为:",getSubnetMasks)
        # 提取网关
        getDefaultGateways.append(adapter.DefaultIPGateway[0])
        print("现在网关为:",getDefaultGateways)
        # 提取dns服务器
        getDNSServers.append(adapter.DNSServerSearchOrder[0])
        print("现在dns服务器为:",getDNSServers)

        '''
        #检测ip是否在线，不可用，需登录
        while True:
            ip2 = random.choice(['216', '217'])
            ip3 = random.randint(1, 254)
            ip4 = random.randint(1, 254)
            newIP = '10.%s.%s.%s' % (ip2, ip3, ip4)
            if self.pingIP(newIP):
                break
              '''
        
        if num == 0:
            arrSubnetMasks =     ['255.255.248.0']      # 子网掩码
            arrDefaultGateways = ['10.202.39.254']      # 网关
            arrDNSServers =      ['114.114.114.114']    # dns服务器
        elif num ==1:
            arrSubnetMasks = getSubnetMasks             # 子网掩码
            arrDefaultGateways = getDefaultGateways     # 网关
            arrDNSServers = getDNSServers               # dns服务器
        else:
            return False
        
        arrGatewayCostMetrics = [1]                 # 这里要设置成1，代表非自动选择


        # 开始执行修改dns
        wayRes = adapter.SetGateways(
            DefaultIPGateway=arrDefaultGateways, GatewayCostMetric=arrGatewayCostMetrics)
        if wayRes[0] == 0:
            print("tip:设置网关成功'")
        else:
            print("tip:修改网关失败: 网关设置发生错误'")
            return False
        dnsRes = adapter.SetDNSServerSearchOrder(
            DNSServerSearchOrder=arrDNSServers)
        if dnsRes[0] == 0:
            print("tip:设置DNS成功,等待3秒刷新缓存'")
            sleep(3)
            # 刷新DNS缓存使DNS生效
            os.system('ipconfig /flushdns')
        else:
            print("tip:修改DNS失败: DNS设置发生错误'")
            return False


        #查询是否接入外网
        for dd in range(1,256):
            print("---开始扫描是否连接到外网---")
            r = run('ping www.baidu.com',
                    stdout=PIPE,
                    stderr=PIPE,
                    stdin=PIPE,
                    shell=True)
                    
            if r.returncode:
                print("开始更改ip")
                """-----------IP变更-------------"""
                # 随机选择了ip的第二段
                # 2022/9/9 只更改组播地址
                if num == 0:
                    ip2 = (ipv4[1])
                    ip3 = (ipv4[2])
                    io4 = (dd)
                elif num == 1:
                    ip2 = random.choice(['200', '202'])
                    ip3 = random.randint(1, 254)        # 随机生成第三段和第四段的值 
                    ip4 = random.randint(1, 254)
                #                
                # ip4 = random.randint(1, 254)  #随机生成值  后期直接遍历
                
                newIP = '10.%s.%s.%s' % (ip2, ip3, ip4)
                # newIP = '10.202.36.%s' % (ip4)

                print("新IP为：",newIP)
                arrIPAddresses = [newIP]  # 设置新的ip
                """-----------IP变更-------------"""

                # 开始执行修改ip、子网掩码、网关
                ipRes = adapter.EnableStatic(
                    IPAddress=arrIPAddresses, SubnetMask=arrSubnetMasks)
                if ipRes[0] == 0:
                    print("tip:设置IP成功")
                    print("设置IP为：%s"% newIP)
                    sleep(5)
                else:
                    if ipRes[0] == 1:
                        print("tip:设置IP成功，需要重启计算机！")
                        return 0
                    else:
                        print("tip:修改IP失败: IP设置发生错误！",ipRes[0])
                        return False

            else:
                print('\033[33m正常联网ok')
                print("IP地址为:",newIP)
                print("子网掩码为:",getSubnetMasks)
                print("网关为:",getDefaultGateways)
                print("DNS服务器为:",getDNSServers)
                print("\033[0m")
                return 0
            # sleep(90) # 每90秒检查一次

    """#ping某ip看是否可以通
    def pingIP(self, ip):
        res = os.popen('ping -n 2 -w 1 %s' % ip).read()  # 内容返回到res
        # res = res.decode('gbk')
        if u'请求超时' in res:  # 注意乱码编码问题
              return False
        else:
              return True
              """
    
    # 测试是否连接到外网
    def ping_BD(self):
        r = run('ping www.baidu.com',
                stdout=PIPE,
                stderr=PIPE,
                stdin=PIPE,
                shell=True)
        if r.returncode:
            # print('relogin 第{}次'.format(cnt))
            return False
        else:
            print('正常联网ok')
            return True

if __name__ == '__main__':
    update = updateIP()
    update.runSet(1)    # 0手动  /  1自动
    #input()
