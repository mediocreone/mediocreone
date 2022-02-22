---
layout: post
title: Cheap VPS CPU benchmark 2022
description: Doing benchmarking using CPU crypto mining 2022
summary: Upcloud and Linode tops as of February 2022.
tags: [CPU_crypto_mining, cheap_VPS, CPU_benchmark]
---

Lately, whatever I built seems to choke on the CPU under stress tests. So I wanted to find a small VPS with persistent CPU performance.  

I used up my [AWS](https://aws.amazon.com/) and [DO](https://www.digitalocean.com/) promotions. [Hetzner](https://www.hetzner.com/) and [GCP](https://cloud.google.com/gcp) were on my initial list. Sadly, my promotional link failed on Hetzner. I kept my GCP credits for NLP related experiments.

My target VPS requirements were at least 2 vCPU and 4GB RAM. I ignored the network bandwidth, virtualization, and disk type.  

 > [Azure free](https://azure.microsoft.com/en-us/free/) ~ 12-month free services and $200 credit expires in a month.  
 > [Alibaba cloud](https://account.alibabacloud.com/register/intl_register.htm) ~ 12-month free services and a free $40 instance expires in a month.  
 > [Linode](https://www.linode.com/) ~ $100$ credit expires in a month.  
 > [Upcloud](https://upcloud.com/) ~ 15$ promotional link + 10$ deposit.  
 > [Vultr](https://www.vultr.com/) ~ $10 referral link + 10$ deposit.  



My objectives were to find out:  
&emsp; 1. What happens if you utilize your VPS CPU 100% consistently?  
&emsp; 2. Which VPS provider currently has the best performing VPS in terms of CPU utilization/cost?  
&emsp; 3. Generally, how performant are these VPS instances compared to my old personal computers?  


So far, I have not done any benchmarking before using a service. I do have some experience comparing apples with bigger or more apples. There may be lots of different ways or tools to do CPU benchmarking. After a bit of googling, I just went with [xmrig](https://xmrig.com/) for the task.

The setup:  
&emsp; Proxy server: Linode Nanode /1c, 1GB/, Ubuntu 20.04 lts + [xmrig-proxy](https://xmrig.com/proxy) connects to mining pools.  
&emsp; Mining pools: [XMR](https://www.getmonero.org/) => [MineXMR](https://minexmr.com/) 1/25-1/27. [SUMO](https://sumokoin.org/) => [HashVault](https://hashvault.pro/) 1/29-1/31. [LTHN](https://www.lt.hn/) => [HashVault](https://hashvault.pro/) 2/3-2-5. [xHV](https://havenprotocol.org/) => [HashVault](https://hashvault.pro/) 2/5-2/7.  
&emsp; VPS/PC setup: Ubuntu 20.04 lts, docker + xmrig connects to the proxy server. VPS(s) are in Frankfurt, DE. PC(s) are in my room.  
&emsp; Old PC(s): **HP** - A broken laptop aged 11+ years, **ASUS** - Desktop processor aged 10+ years. **DELL** - A laptop aged 8+ years. 

The table shows respective VPS/PC CPU info and corresponding hash rates observed on each crypto mining algorithm in ~3 days.

| ------ | ------ | ------ | ------ | ------ | ------ |
| VPS / PC | CPU Model / CPU Caches / Runtime Clock Speed | XMR | xHV | SUMO | LTHN |
| ------: | ------ | ------: | ------: | ------: | ------: |
| Upcloud simple ($20/m) | AMD EPYC 7542 32-Core / L2:1.0MB L3:16.0MB / ~2.89 GHz | 931 | 123 | 117 | 114 |
| Azure D2ads ($91/m) | AMD EPYC 7763 64-Core / L2:0.5MB L3:32.0MB / ~3.26 GHz | 824 | 118 | 92 | 87 |
| Azure D2ds ($99/m) | Intel(R) Xeon(R) Platinum 8370C / L2:1.2MB L3:48.0MB / ~3.49 GHz | 818 | 87 | 84 | 84 |
| Linode shared ($20/m) | AMD EPYC 7642 48-Core / L2:1.0MB L3:32.0MB / ~2.29 GHz | 740 | 113 | 104 | 103 |
| Linode dedicated ($30/m) | AMD EPYC 7642 48-Core / L2:1.0MB L3:32.0MB / ~2.29 GHz | 740 | 110 | 93 | 84 |
| Alibaba ecs.n4.large ($40/m) | Intel(R) Xeon(R) Platinum 8269CY / L2:1.0MB L3:35.8MB / ~2.25 GHz | 455 | 66 | 88 | 73 |
| **ASUS** 400% 200% 400% 400% | Intel(R) Core(TM) i7-2600 @3.40GHz / L2:1.0MB L3:8.0MB / ~3.56 GHz | 1,935 | 130 | 251 | 260 |
| **DELL** 200% 100% 200% 200% | Intel(R) Core(TM) i5-4200U@1.60GHz / L2:0.5MB L3:3.0MB / ~2.17 GHz | 739 | 28 | 63 | 69 |
| **HP** 160% 100% 160% 160% | Intel(R) Core(TM) i5 M 460@2.53GHz / L2:0.5MB L3:3.0MB / ~2.79GHz | 243 | 10 | 21 | 24 |
| | Total combined hash rates | 7,425 | 785 | 913 | 898 |

> Note 1: VPS CPU utilizations limited at 180% during the 3 days on each crypto mining algorithm.  
> Note 2: PC(s) CPU utilization was represented respectively by the crypto mining algorithms (RandomX, CN Heavy, CNR, CNR).

### 1. What happens if you utilize your VPS CPU 100% consistently?  
I utilized all my rented VPS CPU(s) 100% for 3 consecutive days. 

![Alibaba CPU utilization sample](/assets/img/vps/Alibaba-CPU-utilization.png)  
 - Nothing happened with Linode, Upcloud, Alibaba, and Azure. Monitoring screens showed 99-100% CPU usage.
 - Vultr limited my account-wide CPU utilization down to 25% percent. I received an email from Vultr.

<pre>
Vultr:

Based on monitoring results and subsequent analysis, we have determined your CPU resource utilization 
profile is excessive and causing performance issues which may unfairly affect the population of Vultr 
subscribers as a whole. Accordingly, we have limited the maximum CPU resources your instances can consume 
(an account-wide setting). If you are able to reduce the performance impact we have observed, we may be 
able to adjust the limit or remove it altogether.
</pre>

Well, they could have warned me that before limiting my account.

<pre>
Me:

What is the max CPU load that a subscriber is allowed to utilize without affecting the others on Vultr?
</pre>

I hoped that they would give me something like 50%, 75%, or 90%.

<pre>
Vultr:

Our formal service limitations are listed in our acceptable use policy; however, if subscribers exceed 
an acceptable level of usage and results in degrading overall system performance, we have the right 
to limit the account.

We have set resource limits on your account due to high resource utilization that were impacting the 
performance for other customers on the same host node. If you need to make <b>extensive use of the host 
CPU, disk, and/or network resources</b>, please consider using our dedicated (VDC) or bare metal products. 
These alternatives would avoid affecting the performance of other customer's activities.

You may petition to have these limits removed by maintaining consistently lower utilization for <b>two 
weeks</b> and then replying to this ticket.
</pre>

So basically, if you happen to rent a small VPS from Vultr (cloud compute $20/m), then you will likely be limited someday. 
They force account-wide limitations for 14 days with no prior notice. This reminded me of the [incident](https://news.ycombinator.com/item?id=20064169) where [DO](https://www.digitalocean.com/) locked out a startup's access to their rented VPS(s).

### 2. Which VPS provider currently has the best performing VPS in terms of CPU utilization over cost?  
Based on the observed hash rates:  
&emsp; #1 - Upcloud ~ $20/month => 931 h/s  
&emsp; #2 - Linode ~ $20/month => 740 h/s  
&emsp; #3 - Alibaba ~ $40/month => 455 h/s, $80/month => 455 x 2 = 910 h/s  
&emsp; #4 - Azure ~ $91/month => 824 h/s  

Somewhat unexpected findings during the whole experiment:
> Azure VM's `F` series is labeled "Compute Optimized". `D` series is labeled "General purpose". Yet, `D` series had better hash rates.  
> Azure VM's AMD based versions outperformed the Intel based counterparts while being priced cheaper.  
> Linode dedicated VPS ($30/m) instance slightly underperformed the shared ($20/m) instance. Specs were pretty much identical.  
> An Alibaba representative called me asking about my overall experience with their cloud. I did not expect that :)

### 3. Generally, how performant are these VPS instances compared to my old personal computers?  
Now, I feel like my 10-yo ASUS processor is still a beast when it comes to CPU performance.  

---  

&emsp;  

Well, it was a fun experiment for me at least. I am thinking about exploring more crypto stuff when I get a chance.  
PS: 0.0053 [XMR](https://www.getmonero.org/), 0.427 [xHV](https://havenprotocol.org/), 5.14 [SUMO](https://sumokoin.org/), 286.4 [LTHN](https://www.lt.hn/) were mined ~3 days.