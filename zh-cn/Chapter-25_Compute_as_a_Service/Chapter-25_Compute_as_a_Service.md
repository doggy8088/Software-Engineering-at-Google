

**CHAPTER** **25

# Compute as a Service

# 第二十五章 計算即服務

            Written by Onufry Wojtaszczyk Edited by Lisa Carey

*I don’t try to understand computers. I try to understand the programs.*

—Barbara Liskov

After doing the hard work of writing code, you need some hardware to run it. Thus, you go to buy or rent that hardware. This, in essence, is *Compute as a Service* (CaaS), in which “Compute” is shorthand for the computing power needed to actually run your programs.

在完成了編寫程式碼的艱苦工作之後，你需要一些硬體來執行它。因此，你可以購買或租用這些硬體。本質上，這就是“計算即服務”（Compute as a Service，CaaS），其中“計算”是實際執行程式所需的計算能力的簡寫。

This chapter is about how this simple concept—just give me the hardware to run my stuff[^1]—maps into a system that will survive and scale as your organization evolves and grows. It is somewhat long because the topic is complex, and divided into four sections:
- [“Taming the Compute Environment” on page 518 ](#_bookmark2134)covers how Google arrived at its solution for this problem and explains some of the key concepts of CaaS.
- [“Writing Software for Managed Compute” on page 523 ](#_bookmark2156)shows how a managed compute solution affects how engineers write software. We believe that the “cattle, not pets”/flexible scheduling model has been fundamental to Google’s success in the past 15 years and is an important tool in a software engineer’s toolbox.
- [“CaaS Over Time and Scale” on page 530 ](#_bookmark2183)goes deeper into a few lessons Google learned about how various choices about a compute architecture play out as the organization grows and evolves.
- Finally, [“Choosing a Compute Service” on page 535](#_bookmark2202) is dedicated primarily to those engineers who will make a decision about what compute service to use in their organization.

本章講述的是這個簡單的概念--如何為我提供硬體--如何組成一個系統，隨著你的組織的發展和壯大而生存和擴充套件。本章有點長，因為主題很複雜，分為四個部分：
- 第518頁的 "馴服計算環境"涵蓋了谷歌是如何得出這個問題的解決方案的，並解釋了CaaS的一些關鍵概念。

- 第523頁的 "為託管計算編寫軟體"展示了託管計算解決方案如何影響工程師編寫軟體。
- 第523頁的 "為託管計算編寫軟體"展示了託管計算解決方案如何影響工程師編寫軟體。我們相信，"牛，而不是寵物"/靈活的排程模式是谷歌在過去15年成功的根本，也是軟體工程師工具箱中的重要工具。
- 第530頁的 "CaaS隨時間和規模的變化"更深入地探討了谷歌在組織成長和發展過程中對計算架構的各種選擇是如何發揮的一些經驗。
- 最後，第535頁的 "選擇計算服務"主要是獻給那些將決定在其組織中使用何種計算服務的工程師。

> [^1]:	Disclaimer: for some applications, the “hardware to run it” is the hardware of your customers (think, for example, of a shrink-wrapped game you bought a decade ago). This presents very different challenges that we do not cover in this chapter./
> 1   免責宣告：對於某些應用程式，“執行它的硬體”是您客戶的硬體（例如，想想您十年前購買的一款壓縮包裝的遊戲）。這就提出了我們在本章中沒有涉及的非常不同的挑戰。


## Taming the Compute Environment 馴服計算機環境

Google’s internal Borg system[^2] was a precursor for many of today’s CaaS architectures (like Kubernetes or Mesos). To better understand how the particular aspects of such a service answer the needs of a growing and evolving organization, we’ll trace the evolution of Borg and the efforts of Google engineers to tame the compute environment.

谷歌的內部Borg系統是今天許多CaaS架構（如Kubernetes或Mesos）的前身。為了更好地理解這種服務的特定方面如何滿足一個不斷增長和發展的組織的需要，我們將追溯Borg的發展和谷歌工程師為馴服計算環境所做的努力。

### Automation of Toil 自動化操作

Imagine being a student at the university around the turn of the century. If you wanted to deploy some new, beautiful code, you’d SFTP the code onto one of the machines in the university’s computer lab, SSH into the machine, compile and run the code. This is a tempting solution in its simplicity, but it runs into considerable issues over time and at scale. However, because that’s roughly what many projects begin with, multiple organizations end up with processes that are somewhat streamlined evolutions of this system, at least for some tasks—the number of machines grows (so you SFTP and SSH into many of them), but the underlying technology remains. For example, in 2002, Jeff Dean, one of Google’s most senior engineers, wrote the following about running an automated data-processing task as a part of the release process:  
	[Running the task] is a logistical, time-consuming nightmare. It currently requires getting a list of 50+ machines, starting up a process on each of these 50+ machines, and monitoring its progress on each of the 50+ machines. There is no support for automatically migrating the computation to another machine if one of the machines dies, and monitoring the progress of the jobs is done in an ad hoc manner [...] Furthermore, since processes can interfere with each other, there is a complicated, human- implemented “sign up” file to throttle the use of machines, which results in less-than- optimal scheduling, and increased contention for the scarce machine resources

想象一下，在世紀之交的時候，你是一個大學的學生。如果你想部署一些新的、厲害的程式碼，你會把程式碼從SFTP複製到大學計算機實驗室的一臺機器上，SSH進入機器，編譯並執行程式碼。這是一個簡單而誘人的解決方案，但隨著時間的推移和規模的擴大，它遇到了相當多的問題。然而，因為這大概是許多專案開始時的情況，多個組織最終採用的流程在某種程度上是這個系統的流程演變，至少對於某些任務來說是這樣的--機器的數量增加了（所以你SFTP和SSH進入其中許多機器），但底層技術仍然存在。例如，2002年，谷歌最資深的工程師之一傑夫·迪恩（Jeff Dean）寫了以下關於在發佈過程中執行自動資料處理任務的文章：  
	[執行任務]是一個組織管理的、耗時的噩夢。目前，它需要獲得一個50多臺機器的列表，在這50多臺機器上各啟動一個程序，並在這50多臺機器上各監控其進度。如果其中一臺機器宕機了，不支援自動將計算遷移到另一臺機器上，而且監測工作的進展是以臨時的方式進行的[......]此外，由於程序可以相互干擾，有一個複雜的、人工實現的 "註冊 "檔案來節制機器的使用，這導致了非最優排程，增加了對稀缺機器資源的爭奪。	

This was an early trigger in Google’s efforts to tame the compute environment, which explains well how the naive solution becomes unmaintainable at larger scale.

這是谷歌努力馴服計算環境的早期導火索，這很好地解釋了這種幼稚的解決方案如何在更大範圍內變得不可維護。

> [^2]:	Abhishek Verma, Luis Pedrosa, Madhukar R Korupolu, David Oppenheimer, Eric Tune, and John Wilkes, “Large-scale cluster management at Google with Borg,” EuroSys, Article No.: 18 (April 2015): 1–17.
>[^2]:   Abhishek Verma、Luis Pedrosa、Madhukar R Korupolu、David Oppenheimer、Eric Tune和John Wilkes，“谷歌與Borg的大規模叢集管理”，EuroSys，文章編號：18（2015年4月）：1-17。


#### Simple automations 簡單自動化

There are simple things that an organization can do to mitigate some of the pain. The process of deploying a binary onto each of the 50+ machines and starting it there can easily be automated through a shell script, and then—if this is to be a reusable solution—through a more robust piece of code in an easier-to-maintain language that will perform the deployment in parallel (especially since the “50+” is likely to grow over time).

一個組織可以做一些簡單的事情來減輕一些痛苦。將二進位制檔案部署到50多臺機器上並在其中啟動的過程可以透過一個shell指令碼輕鬆實現自動化，如果這是一個可重用的解決方案，則可以透過一段更健壯的程式碼，使用一種更易於維護的語言，並行執行部署（特別是因為“50+”可能會隨著時間的推移而增長）。

More interestingly, the monitoring of each machine can also be automated. Initially, the person in charge of the process would like to know (and be able to intervene) if something went wrong with one of the replicas. This means exporting some monitoring metrics (like “the process is alive” and “number of documents processed”) from the process—by having it write to a shared storage, or call out to a monitoring service, where they can see anomalies at a glance. Current open source solutions in that space are, for instance, setting up a dashboard in a monitoring tool like Graphana or Prometheus.

更有趣的是，對每臺機器的監控也可以自動化。最初，負責程序的人希望知道（並能夠進行干預），如果其中一個副本出了問題。這意味著從程序中輸出一些監控指標（如 "程序是活的 "和 "處理的檔案數"）--讓它寫到一個共享儲存中，或呼叫一個監控服務，在那裡他們可以一眼看到異常情況。目前該領域的開源解決方案是，例如，在Graphana或Prometheus等監控工具中設定一個儀表盤。

If an anomaly is detected, the usual mitigation strategy is to SSH into the machine, kill the process (if it’s still alive), and start it again. This is tedious, possibly error prone (be sure you connect to the right machine, and be sure to kill the right process), and could be automated:
- Instead of manually monitoring for failures, one can use an agent on the machine that detects anomalies (like “the process did not report it’s alive for the past five minutes” or “the process did not process any documents over the past 10 minutes”), and kills the process if an anomaly is detected.
- Instead of logging in to the machine to start the process again after death, it might be enough to wrap the whole execution in a “while true; do run && break; done” shell script.

如果檢測到異常，通常的緩解策略是透過SSH進入機器，殺死程序（如果它還活著），然後重新啟動它。這很繁瑣，可能容易出錯（要確保你連線到正確的機器，並確保殺死正確的程序），並且可以自動化：
- 與其手動監控故障，不如在機器上使用一個代理，檢測異常情況（比如 "該程序在過去5分鐘內沒有報告它處於”活動狀態"或 "該程序在過去10分鐘內沒有處理任何檔案"），如果檢測到異常情況，就殺死該程序。
- 與其在宕掉後登入到機器上再次啟動程序，不如將整個執行過程包裹在一個 "while true; do run && break; done "的shell指令碼中。

The cloud world equivalent is setting an autohealing policy (to kill and re-create a VM or container after it fails a health check).

在雲端計算世界中，相當於設定了一個自動修復策略（在執行狀況檢查失敗後殺死並重新建立VM或容器）。

These relatively simple improvements address a part of Jeff Dean’s problem described earlier, but not all of it; human-implemented throttling, and moving to a new machine, require more involved solutions.

這些相對簡單的改進解決了前面描述的傑夫·迪恩問題的一部分，但不是全部；人工實現的流程，以及轉移到新機器，需要更復雜的解決方案。

#### Automated scheduling 自動排程

The natural next step is to automate machine assignment. This requires the first real “service” that will eventually grow into “Compute as a Service.” That is, to automate scheduling, we need a central service that knows the complete list of machines available to it and can—on demand—pick a number of unoccupied machines and automatically deploy your binary to those machines. This eliminates the need for a hand-maintained “sign-up” file, instead delegating the maintenance of the list of machines to computers. This system is strongly reminiscent of earlier time-sharing architectures.

下一步自然是自動化機器資源分配。這需要第一個真正的“服務”，最終將發展為“計算即服務”。也就是說，為了自動化排程，我們需要一箇中心服務，它知道可用機器的完整列表，並且可以根據需要選擇一些未佔用的機器，並自動將二進位制檔案部署到這些機器上。這樣就不需要人動了維護“註冊”檔案，而不是將機器列表的維護委託給計算機。該系統強烈地讓人想起早期的分時體系結構。

A natural extension of this idea is to combine this scheduling with reaction to machine failure. By scanning machine logs for expressions that signal bad health (e.g., mass disk read errors), we can identify machines that are broken, signal (to humans) the need to repair such machines, and avoid scheduling any work onto those machines in the meantime. Extending the elimination of toil further, automation can try some fixes first before involving a human, like rebooting the machine, with the hope that whatever was wrong goes away, or running an automated disk scan.

這種想法的自然延伸是將這種排程與對機器故障的反應結合起來。透過掃描機器日誌以查詢表示執行狀況不良的指標（例如，大量磁碟讀取錯誤），我們可以識別出損壞的機器，向（工程師）發出修復此類別機器的訊號，同時避免將任何工作安排到這些機器上。為了進一步消除繁重的工作，自動化可以在人工干預之前先嚐試一些修復，比如重新啟動機器，希望任何錯誤都能消失，或者執行自動磁碟掃描。

One last complaint from Jeff ’s quote is the need for a human to migrate the computation to another machine if the machine it’s running on breaks. The solution here is simple: because we already have scheduling automation and the capability to detect that a machine is broken, we can simply have the scheduler allocate a new machine and restart the work on this new machine, abandoning the old one. The signal to do this might come from the machine introspection daemon or from monitoring of the individual process.

Jeff參考的最後一個抱怨是，如果正在執行的機器出現故障，人們需要將計算機遷移到另一臺機器上。這裡的解決方案很簡單：因為我們已經有了排程自動化和檢測機器故障的能力，我們可以簡單地讓排程器分配一臺新機器，並在這臺新機器上重新啟動工作，放棄舊機器。執行此操作的訊號可能來自機器內部守護程序或來自於對單個程序的監控。

All of these improvements systematically deal with the growing scale of the organization. When the fleet was a single machine, SFTP and SSH were perfect solutions, but at the scale of hundreds or thousands of machines, automation needs to take over. The quote we started from came from a 2002 design document for the “Global WorkQueue,” an early CaaS internal solution for some workloads at Google.

所有這些改進都系統地處理了組織規模不斷擴大的問題。當叢集是一臺機器時，SFTP和SSH是完美的解決方案，但在數百或數千臺機器的規模上，需要自動化來接管。我們所參考的這句話來自2002年 "全球工作佇列 "的設計檔案，這是Google早期針對某些工作負載的CaaS內部解決方案。

### Containerization and Multitenancy 容器化和多租戶

So far, we implicitly assumed a one-to-one mapping between machines and the programs running on them. This is highly inefficient in terms of computing resource (RAM, CPU) consumption, in many ways:
- It’s very likely to have many more different types of jobs (with different resource requirements) than types of machines (with different resource availability), so many jobs will need to use the same machine type (which will need to be provisioned for the largest of them).
- Machines take a long time to deploy, whereas program resource needs grow over time. If obtaining new, larger machines takes your organization months, you need to also make them large enough to accommodate expected growth of resource needs over the time needed to provision new ones, which leads to waste, as new machines are not utilized to their full capacity.[^3]
- Even when the new machines arrive, you still have the old ones (and it’s likely wasteful to throw them away), and so you must manage a heterogeneous fleet that does not adapt itself to your needs.

到目前為止，我們隱式地假設機器和執行在機器上的程式之間存在一對一的對映。在計算資源（RAM、CPU）消耗方面，這在許多方面都是非常低效的：

- 與機器型別（具有不同的資源可用性）相比，它很可能有更多不同型別的作業（具有不同的資源需求），因此許多作業將需要使用相同的機器型別（需要為最大的機器型別提供）。
- 機器的部署需要很長時間，而程式資源需要隨著時間的推移而增長。如果獲得新的、更大的機器需要花費你的組織幾個月的時間，你還需要使它們足夠大，以適應在提供新機器所需時間內資源需求的預期增長，這會導致浪費，因為新機器沒有充分利用其容量。
- 即使新機器到了，你還有舊機器（扔掉它們很可能是浪費），因此你必須管理一個不適應你需求的異構叢集。

The natural solution is to specify, for each program, its resource requirements (in terms of CPU, RAM, disk space), and then ask the scheduler to bin-pack replicas of the program onto the available pool of machines.

最自然的解決方案是為每個程式指定其資源需求（CPU、RAM、磁碟空間），然後要求排程器將程式的副本打包到可用的機器資源池中。

> [^3]:	Note that this and the next point apply less if your organization is renting machines from a public cloud provider./
> 3 請注意，如果你的組織從公共雲提供商那裡租用機器，這一點和下一點就不適用。

#### My neighbor’s dog barks in my RAM 鄰居家的狗在我的記憶體中吠叫

The aforementioned solution works perfectly if everybody plays nicely. However, if I specify in my configuration that each replica of my data-processing pipeline will consume one CPU and 200 MB of RAM, and then—due to a bug, or organic growth—it starts consuming more, the machines it gets scheduled onto will run out of resources. In the CPU case, this will cause neighboring serving jobs to experience latency blips; in the RAM case, it will either cause out-of-memory kills by the kernel or horrible latency due to disk swap.[^4]

如果每個人都能很好地發揮，上述的解決方案就能完美地工作。然而，如果我在配置中指定我的資料處理管道的每個副本將佔用一個CPU和200MB的記憶體，然後由於一個錯誤，或指數式增長，它開始消耗更多的資源，那麼它排程到的機器將耗盡資源。在消耗CPU的情況下，這將導致相鄰的服務工作出現延遲；在消耗RAM的情況下，它要麼會導致核心記憶體不足，要麼會由於磁碟交換而導致可怕的延遲。

Two programs on the same computer can interact badly in other ways as well. Many programs will want their dependencies installed on a machine, in some specific version—and these might collide with the version requirements of some other program. A program might expect certain system-wide resources (think about /tmp) to be available for its own exclusive use. Security is an issue—a program might be handling sensitive data and needs to be sure that other programs on the same machine cannot access it.

同一臺計算機上的兩個程式在其他方面也會相互影響。許多程式希望在特定版本的計算機上安裝它們的依賴項，這些依賴項可能會與其他程式的版本要求發生衝突。一個程式可能期望某些系統範圍的資源（想想/tmp）可供自己專用。安全性是一個問題--程式可能正在處理敏感資料，需要確保同一臺計算機上的其他程式無法訪問它。

Thus, a multitenant compute service must provide a degree of *isolation,* a guarantee of some sort that a process will be able to safely proceed without being disturbed by the other tenants of the machine.

因此，多租戶計算服務必須提供一定程度的*隔離，*某種程度上保證一個程序能夠安全進行而不被機器的其他租戶干擾。

A classical solution to isolation is the use of virtual machines (VMs). These, however, come with significant overhead[^5] in terms of resource usage (they need the resources to run a full operating system inside) and startup time (again, they need to boot up a full operating system). This makes them a less-than-perfect solution for batch job containerization for which small resource footprints and short runtimes are expected. This led Google’s engineers designing Borg in 2003 to look to different solutions, ending up with *containers—*a lightweight mechanism based on cgroups (contributed by Google engineers into the Linux kernel in 2007) and chroot jails, bind mounts and/or union/overlay filesystems for filesystem isolation. Open source container implementations include Docker and LMCTFY.

隔離的一個經典解決方案是使用虛擬機器（VM）。然而，這些虛擬機器在資源使用（它們需要資源在裡面執行一個完整的作業系統）和啟動時間（同樣，它們需要啟動一個完整的作業系統）方面有很大的開銷。這使得它們成為一個不太完美的解決方案，使用於資源佔用少、執行時間短的批量作業容器化。這導致谷歌在2003年設計Borg的工程師們尋找不同的解決方案，最終找到了*容器*--一種基於cgroups（由谷歌工程師在2007年貢獻給Linux核心）和chroot jails、bind mounts和/或union/overlay檔案系統進行檔案系統隔離的輕型機制。開源容器的實現包括Docker和LMCTFY。

Over time and with the evolution of the organization, more and more potential isolation failures are discovered. To give a specific example, in 2011, engineers working on Borg discovered that the exhaustion of the process ID space (which was set by default to 32,000 PIDs) was becoming an isolation failure, and limits on the total number of processes/threads a single replica can spawn had to be introduced. We look at this example in more detail later in this chapter.

隨著時間的推移和組織的發展，發現了越來越多的潛在隔離故障。舉個具體的例子，2011年，在Borg工作的工程師發現，程序ID空間（預設設定為32000個PID）的耗盡正在成為一個隔離故障，因此不得不引入對單個副本可產生的程序/執行緒總數的限制。我們將在本章後面更詳細地討論這個例子。

> [^4]:	Google has chosen, long ago, that the latency degradation due to disk swap is so horrible that an out-of- memory kill and a migration to a different machine is universally preferable—so in Google’s case, it’s always an out-of-memory kill./
> 4 谷歌很久以前就確認了，由於磁碟交換導致的延遲降低是如此可怕，以至於記憶體不足殺死和遷移到另一臺機器是普遍可取的，因此在谷歌的情況下，總是記憶體不足殺死程序。
> 
> [^5]:	Although a considerable amount of research is going into decreasing this overhead, it will never be as low as a process running natively./
5 儘管有大量的研究正在致力於減少這種開銷，但它永遠不會像一個本機執行的程序那麼低。

#### Rightsizing and autoscaling 合理調整和自動縮放

The Borg of 2006 scheduled work based on the parameters provided by the engineer in the configuration, such as the number of replicas and the resource requirements.

2006年的Borg根據工程師在配置中提供的引數，如複製的數量和資源要求，進行工作。

Looking at the problem from a distance, the idea of asking humans to determine the resource requirement numbers is somewhat flawed: these are not numbers that humans interact with daily. And so, these configuration parameters become themselves, over time, a source of inefficiency. Engineers need to spend time determining them upon initial service launch, and as your organization accumulates more and more services, the cost to determine them scales up. Moreover, as time passes, the program evolves (likely grows), but the configuration parameters do not keep up. This ends in an outage—where it turns out that over time the new releases had resource requirements that ate into the slack left for unexpected spikes or outages, and when such a spike or outage actually occurs, the slack remaining turns out to be insufficient.

從遠處看這個問題，要求人類確定資源需求數字的想法有些缺陷：這些數字不是人類每天與之互動的數字。因此，隨著時間的推移，這些配置引數本身就成為效率低下的來源。工程師需要花時間在最初的服務啟動時確定這些引數，而隨著你的組織積累越來越多的服務，確定這些引數的成本也在增加。此外，隨著時間的推移，程式的發展（可能會增長），但配置引數並沒有跟上。這最終導致了故障的發生--事實證明，隨著時間的推移，新版本的資源需求吃掉了留預期外高峰或故障的容災空間，而當這種高峰或故障實際發生時，剩餘的容災空間被證明是不夠的。

The natural solution is to automate the setting of these parameters. Unfortunately, this proves surprisingly tricky to do well. As an example, Google has only recently reached a point at which more than half of the resource usage over the whole Borg fleet is determined by rightsizing automation. That said, even though it is only half of the usage, it is a larger fraction of configurations, which means that the majority of engineers do not need to concern themselves with the tedious and error-prone burden of sizing their containers. We view this as a successful application of the idea that “easy things should be easy, and complex things should be possible”—just because some fraction of Borg workloads is too complex to be properly managed by rightsizing doesn’t mean there isn’t great value in handling the easy cases.

自然的解決方案是將這些引數的設定自動化。不幸的是，要做好這件事非常棘手。作為一個例子，谷歌最近才達到一個點，即整個Borg叢集超過一半的資源使用是由調整大小有自動化系統決定的。也就是說，儘管這只是一半的使用量，但它是配置中較大的一部分，這意味著大多數工程師不需要擔心確定容器大小的繁瑣且容易出錯的問題。我們認為這是對 "簡單的事情應該是容易的，複雜的事情應該是可能的 "這一理念的成功應用--僅僅因為Borg工作負載的某些部分過於複雜，無法透過許可權調整進行適當管理，並不意味著在處理簡單情況時沒有很大的價值。

### Summary 總結

As your organization grows and your products become more popular, you will grow in all of these axes:
- Number of different applications to be managed
- Number of copies of an application that needs to run
- The size of the largest application

隨著你的組織的發展和產品的普及，你將在所有這些軸上成長：
- 需要管理的不同應用程式的數量
- 需要執行的應用程式的副本數量
- 最大的應用程式的規模

To effectively manage scale, automation is needed that will enable you to address all these growth axes. You should, over time, expect the automation itself to become more involved, both to handle new types of requirements (for instance, scheduling for GPUs and TPUs is a major change in Borg that happened over the past 10 years) and increased scale. Actions that, at a smaller scale, could be manual, will need to be automated to avoid a collapse of the organization under the load.

為了有效地管理規模，需要自動化，使你能夠解決所有這些增長軸。隨著時間的推移，你應該期待自動化本身變得更多，既要處理新型別的要求（例如，GPU和TPU的排程是 Borg 在過去10年裡發生的一個主要變化），又要處理規模的增加。在較小的規模下，可能是手動的操作，將需要自動化，以避免組織在負載下的崩潰。

One example—a transition that Google is still in the process of figuring out—is automating the management of our *datacenters*. Ten years ago, each datacenter was a separate entity. We manually managed them. Turning a datacenter up was an involved manual process, requiring a specialized skill set, that took weeks (from the moment when all the machines are ready) and was inherently risky. However, the growth of the number of datacenters Google manages meant that we moved toward a model in which turning up a datacenter is an automated process that does not require human intervention.

一個例子--谷歌仍在摸索的過渡--是自動管理我們的*資料中心*。十年前，每個資料中心是一個獨立的實體。我們手動管理它們。啟用一個數據中心是一個複雜的手動過程，需要專門的技能，需要幾周的時間（從所有機器準備好的那一刻開始），而且本身就有風險。然而，谷歌管理的資料中心數量的增長意味著我們轉向了一種模式，即啟動資料中心是一個不需要人工干預的自動化過程。

## Writing Software for Managed Compute 為管理計算能力編寫軟體

The move from a world of hand-managed lists of machines to the automated scheduling and rightsizing made management of the fleet much easier for Google, but it also took profound changes to the way we write and think about software.

從手工管理的機器列表轉向自動化的計劃和調整規模，這使得谷歌更容易管理機隊，但也給我們編寫和思考軟體的方式帶來了深刻的變化。

### Architecting for Failure 故障架構

Imagine an engineer is to process a batch of one million documents and validate their correctness. If processing a single document takes one second, the entire job would take one machine roughly 12 days—which is probably too long. So, we shard the work across 200 machines, which reduces the runtime to a much more manageable 100 minutes.

想象一下，一個工程師要處理一批100萬份檔案並驗證其正確性。如果處理一個檔案需要一秒鐘，那麼整個工作將需要一臺機器大約12天--這可能太長了。因此，我們把工作分散到200臺機器上，這將執行時間減少到更易於管理的100分鐘。

As discussed in [“Automated scheduling” on page 519](#_bookmark2140), in the Borg world, the scheduler can unilaterally kill one of the 200 workers and move it to a different machine.[^6] The “move it to a different machine” part implies that a new instance of your worker can be stamped out automatically, without the need for a human to SSH into the machine and tune some environment variables or install packages.

正如第519頁 "自動排程 "中所討論的，在博格世界中，排程中心可以單方面殺死200個worker中的一個，並把它移到不同的機器上。"把它移到不同的機器上 "這部分意味著你的workers的新實例可以自動被輸出出來，不需要人手動去SSH進入機器，調整一些環境變數或安裝軟體套件。

The move from “the engineer has to manually monitor each of the 100 tasks and attend to them if broken” to “if something goes wrong with one of the tasks, the system is architected so that the load is picked up by others, while the automated scheduler kills it and reinstantiates it on a new machine” has been described many years later through the analogy of “pets versus cattle.”[^7]

從 "工程師必須手動監控100個任務中的每一個，並在出現問題時對其進行處理 "到 "如果其中一個任務出現問題，系統會被設計成由其他任務來承擔，而自動化系統會將其殺死並在新的機器上重新執行"，這一轉變在許多年後透過 "寵物與牛 "的比喻來描述。

If your server is a pet, when it’s broken, a human comes to look at it (usually in a panic), understand what went wrong, and hopefully nurse it back to health. It’s difficult to replace. If your servers are cattle, you name them replica001 to replica100, and if one fails, automation will remove it and provision a new one in its place. The distinguishing characteristic of “cattle” is that it’s easy to stamp out a new instance of the job in question—it doesn’t require manual setup and can be done fully automatically. This allows for the self-healing property described earlier—in the case of a failure, automation can take over and replace the unhealthy job with a new, healthy one without human intervention. Note that although the original metaphor spoke of servers (VMs), the same applies to containers: if you can stamp out a new version of the container from an image without human intervention, your automation will be able to autoheal your service when required.

如果你的伺服器是一隻寵物，當它壞了時，一個人會來看它（通常是驚慌失措），瞭解出了什麼問題，並希望護理它恢復健康。很難更換。如果您的伺服器是牛，您可以將它們命名為replica001到replica100，如果其中一個伺服器出現故障，自動化將刪除它並在其位置提供一個新的伺服器。“牛群”的獨特之處在於，它可以很容易地刪除相關作業的新實例--它不需要手動設定，可以完全自動完成。這就實現了前面描述的自愈特性。在發生故障的情況下，自動化可以接管不健康的工作，並用一個新的、健康的工作替換它，而無需人工干預。請注意，儘管最初的隱喻談到了伺服器（VM），但同樣適用於容器：如果你可以在無需人工干預的情況下從映像中刪除容器的新版本，那麼你的自動化將能夠在需要時自動修復您的服務。

If your servers are pets, your maintenance burden will grow linearly, or even superlinearly, with the size of your fleet, and that’s a burden that no organization should accept lightly. On the other hand, if your servers are cattle, your system will be able to return to a stable state after a failure, and you will not need to spend your weekend nursing a pet server or container back to health.

如果你的伺服器是寵物，你的維護負擔將隨著你的叢集規模線性增長，甚至是超線性增長，這是任何組織都不應輕視的負擔。另一方面，如果你的伺服器是牛，你的系統將能夠在故障後恢復到一個穩定的狀態，你將不需要花週末的時間來護理一個寵物伺服器或容器恢復健康。

Having your VMs or containers be cattle is not enough to guarantee that your system will behave well in the face of failure, though. With 200 machines, one of the replicas being killed by Borg is quite likely to happen, possibly more than once, and each time it extends the overall duration by 50 minutes (or however much processing time was lost). To deal with this gracefully, the architecture of the processing needs to be different: instead of statically assigning the work, we instead divide the entire set of one million documents into, say, 1,000 chunks of 1,000 documents each. Whenever a worker is finished with a particular chunk, it reports the results, and picks up another. This means that we lose at most one chunk of work on a worker failure, in the case when the worker dies after finishing the chunk, but before reporting it. This, fortunately, fits very well with the data-processing architecture that was Google’s standard at that time: work isn’t assigned equally to the set of workers at the start of the computation; it’s dynamically assigned during the overall processing in order to account for workers that fail.

不過，讓虛擬機器或容器正常執行並不足以保證系統在出現故障時表現良好。對於200臺機器，Borg很可能會殺死其中一個複製副本，可能不止一次，每次都會將整個持續時間延長50分鐘（或者無論損失多少處理時間）。為了優雅地處理這個問題，處理的架構需要改變：我們不是固定地分配工作，而是將100萬個文件的整個集合劃分為1000個塊，每個塊包含1000個文件。每當一個worker完成了一個特定的塊，它就會報告結果，並拿起另一個。這意味著，如果worker在完成區塊後但在報告之前宕機，我們在worker失敗時最多損失一個區塊的工作。幸運的是，這非常符合當時谷歌標準的資料處理架構：在計算開始時，任務並不是平均分配給一組worker的；而是在整個處理過程中動態分配的，以便考慮到worker的失敗。

Similarly, for systems serving user traffic, you would ideally want a container being rescheduled not resulting in errors being served to your users. The Borg scheduler, when it plans to reschedule a container for maintenance reasons, signals its intent to the container to give it notice ahead of time. The container can react to this by refusing new requests while still having the time to finish the requests it has ongoing. This, in turn, requires the load-balancer system to understand the “I cannot accept new requests” response (and redirect traffic to other replicas).

同樣，對於服務於使用者流量的系統來說，理想情況下，希望容器排程不會導致向用戶提供錯誤。當Borg排程器由於維護原因計劃重新排程一個容器時，會向容器發出訊號，提前通知它的意圖。容器可以透過拒絕新的請求來做出反應，同時還有時間來完成它正在進行的請求。這反過來要求負載均衡器系統理解 "我不能接受新請求 "的響應（並將流量重新導向到其他副本）。

To summarize: treating your containers or servers as cattle means that your service can get back to a healthy state automatically, but additional effort is needed to make sure that it can function smoothly while experiencing a moderate rate of failures.

總而言之：將容器或伺服器視為“牛”意味著你的服務可以自動恢復到正常狀態，但還需要付出額外的努力，以確保它能夠在遇到中等故障率的情況下順利執行。

>  [^6]:	The scheduler does not do this arbitrarily, but for concrete reasons (like the need to update the kernel, or a disk going bad on the machine, or a reshuffle to make the overall distribution of workloads in the datacenter bin-packed better). However, the point of having a compute service is that as a software author, I should neither know nor care why regarding the reasons this might happen./
>  6 排程器並不是隨意這樣做的，而是出於具體的原因（比如需要更新核心，或者機器上的磁碟壞了，或者為了更好地打包資料中心容器中的工作負載的總體分佈而進行的改組）。然而，擁有計算服務的意義在於，作為軟體作者，我不應該知道也不關心為什麼會發生這種情況。
>
> [^7]:	The “pets versus cattle” metaphor is attributed to Bill Baker by Randy Bias and it’s become extremely popular as a way to describe the “replicated software unit” concept. As an analogy, it can also be used to describe concepts other than servers; for example, see Chapter 22./ 
>  7 "寵物與牛 "的比喻是由Randy Bias歸功於Bill Baker的，它作為描述 "複製的軟體單元 "概念的一種方式，已經變得非常流行。作為一個比喻，它也可以用來描述伺服器以外的概念；例如，見第22章。

### Batch Versus Serving 批量作業與服務作業

The Global WorkQueue (which we described in the first section of this chapter) addressed the problem of what Google engineers call “batch jobs”—programs that are expected to complete some specific task (like data processing) and that run to completion. Canonical examples of batch jobs would be logs analysis or machine learning model learning. Batch jobs stood in contrast to “serving jobs”—programs that are expected to run indefinitely and serve incoming requests, the canonical example being the job that served actual user search queries from the prebuilt index.

全域性工作佇列（Global WorkQueue）（我們在本章第一節中描述過）解決了谷歌工程師所說的 "批處理作業 "的問題--這些程式要完成一些特定的任務（如資料處理），並且要執行到完成。批量作業的典型例子是日誌分析或機器學習模型學習。批量作業與 "服務作業 "形成鮮明對比--這些程式預計將無限期地執行並為傳入的請求提供服務，典型的例子是為來自預建構索引的實際使用者搜尋查詢提供服務的作業。

These two types of jobs have (typically) different characteristics,[^8] in particular:
- Batch jobs are primarily interested in throughput of processing. Serving jobs care about latency of serving a single request.
- Batch jobs are short lived (minutes, or at most hours). Serving jobs are typically long lived (by default only restarted with new releases).
- Because they’re long lived, serving jobs are more likely to have longer startup times.

這兩類別作業（通常）具有不同的特點，特別是：
- 批量作業主要關心的是處理的吞吐量。服務作業關心的是服務單個請求的延遲。
- 批量作業的生命週期很短（幾分鐘，或最多幾個小時）。服務工作通常是長期存在的（預設情況下，只有在新版本發佈時才會重新啟動）。
- 因為它們是長期存在的，所以服務工作更有可能有較長的啟動時間。

So far, most of our examples were about batch jobs. As we have seen, to adapt a batch job to survive failures, we need to make sure that work is spread into small chunks and assigned dynamically to workers. The canonical framework for doing this at Google was MapReduce,[^9] later replaced by Flume.[^10]

到目前為止，我們大部分的例子都是關於批處理作業的。正如我們所看到的，為了使批處理作業適應失敗，我們需要確保工作被分散成小塊，並動態地分配給worker。在谷歌，這樣做的典型框架是MapReduce，後來被Flume取代。

Serving jobs are, in many ways, more naturally suited to failure resistance than batch jobs. Their work is naturally chunked into small pieces (individual user requests) that are assigned dynamically to workers—the strategy of handling a large stream of requests through load balancing across a cluster of servers has been used since the early days of serving internet traffic.

在許多方面，服務作業比批量作業更自然地適合於抗故障。他們的工作自然地分成小塊（單個使用者請求），動態地分配給worker。從網際網路流量服務的早期開始，就採用了透過伺服器叢集負載平衡來處理大量請求的策略。

However, there are also multiple serving applications that do not naturally fit that pattern. The canonical example would be any server that you intuitively describe as a “leader” of a particular system. Such a server will typically maintain the state of the system (in memory or on its local filesystem), and if the machine it is running on goes down, a newly created instance will typically be unable to re-create the system’s state. Another example is when you have large amounts of data to serve—more than fits on one machine—and so you decide to shard the data among, for instance, 100 servers, each holding 1% of the data, and handling requests for that part of the data. This is similar to statically assigning work to batch job workers; if one of the servers goes down, you (temporarily) lose the ability to serve a part of your data. A final example is if your server is known to other parts of your system by its hostname. In that case, regardless of how your server is structured, if this specific host loses network connectivity, other parts of your system will be unable to contact it.[^11]

然而，也有多個服務應用程式不適合這種模式。最典型的例子是你直觀地描述為特定系統的“領導者”的任何伺服器。這樣的伺服器通常會維護系統的狀態（在記憶體中或在其本地檔案系統中），如果它所執行的機器出現故障，新建立的實例通常無法重新建立系統的狀態。另一個例子是，當你有大量的資料需要服務--超過一臺機器所能容納的--於是你決定將資料分片，比如說，100臺伺服器，每臺都持有1%的資料，並處理這部分資料的請求。這類似於將工作靜態地分配給批處理工作的worker；如果其中一個伺服器發生故障，你就會（暫時）失去為部分資料服務的能力。最後一個示例是，系統的其他部分是否知道伺服器的主機名。在這種情況下，無論伺服器的結構如何，如果此特定主機失去網路連線，系統的其他部分將無法與之聯絡。


> [^8]:	Like all categorizations, this one isn’t perfect; there are types of programs that don’t fit neatly into any of the categories, or that possess characteristics typical of both serving and batch jobs. However, like most useful categorizations, it still captures a distinction present in many real-life cases./
> 8 像所有的分類一樣，這個分類並不完美；有些型別的程式不適合任何類別，或者具有服務作業和批處理作業的典型特徵。然而，與最有用的分類一樣，它仍然抓住了許多實際案例中存在的區別。

> [^9]:	See Jeffrey Dean and Sanjay Ghemawat, “MapReduce: Simplified Data Processing on Large Clusters,” 6th Symposium on Operating System Design and Implementation (OSDI), 2004./
> 9 見Jeffrey Dean和Sanjay Ghemawat，"MapReduce。簡化大型叢集上的資料處理，"第六屆作業系統設計與實現研討會（OSDI），2004。
>
> [^10]:	Craig Chambers, Ashish Raniwala, Frances Perry, Stephen Adams, Robert Henry, Robert Bradshaw, and Nathan Weizenbaum, “Flume‐Java: Easy, Efficient Data-Parallel Pipelines,” ACM SIGPLAN Conference on Programming Language Design and Implementation (PLDI), 2010./
> 10 Craig Chambers, Ashish Raniwala, Frances Perry, Stephen Adams, Robert Henry, Robert Bradshaw, and Nathan Weizenbaum, "Flume-Java: Easy, Efficient Data-Parallel Pipelines," ACM SIGPLAN程式語言設計與實現會議（PLDI），2010。
>
> [^11]:	See also Atul Adya et al. “Auto-sharding for datacenter applications,” OSDI, 2019; and Atul Adya, Daniel Myers, Henry Qin, and Robert Grandl, “Fast key-value stores: An idea whose time has come and gone,” HotOS XVII, 2019./
> 11 另見Atul Adya等人，"資料中心應用的自動分片"，OSDI，2019；以及Atul Adya、Daniel Myers、Henry Qin和Robert Grandl，"快速鍵值儲存。一個時代已經到來的想法，" HotOS XVII，2019年。


### Managing State 管理狀態

One common theme in the previous description focused on *state* as a source of issues when trying to treat jobs like cattle.[^12] Whenever you replace one of your cattle jobs, you lose all the in-process state (as well as everything that was on local storage, if the job is moved to a different machine). This means that the in-process state should be treated as transient, whereas “real storage” needs to occur elsewhere.

在前面的描述中，有一個共同的主題集中在*狀態*上，當試影象對待牛一樣對待作業時，*狀態*是問題的來源。每當你替換你的一個牛的作業時，你會失去所有的程序中的狀態（以及所有在本地儲存的東西，如果作業被轉移到不同的機器上）。這意味著程序內狀態應被視為瞬態，而“真實儲存”需要發生在其他地方。

The simplest way of dealing with this is extracting all storage to an external storage system. This means that anything that should survive past the scope of serving a single request (in the serving job case) or processing one chunk of data (in the batch case) needs to be stored off machine, in durable, persistent storage. If all your local state is immutable, making your application failure resistant should be relatively painless.

處理這個問題的最簡單方法是將所有的儲存提取到外部儲存系統。這意味著任何應該在服務單一請求（在服務工作的情況下）或處理一個數據塊（在批處理的情況下）的範圍內生存的東西都需要儲存在機器外的永續性儲存中。如果所有的本地狀態都是不可變的，那麼讓應用程式具有抗故障能力應該是相對容易的。

Unfortunately, most applications are not that simple. One natural question that might come to mind is, “How are these durable, persistent storage solutions implemented— are *they* cattle?” The answer should be “yes.” Persistent state can be managed by cattle through state replication. On a different level, RAID arrays are an analogous concept; we treat disks as transient (accept the fact one of them can be gone) while still maintaining state. In the servers world, this might be realized through multiple replicas holding a single piece of data and synchronizing to make sure every piece of data is replicated a sufficient number of times (usually 3 to 5). Note that setting this up correctly is difficult (some way of consensus handling is needed to deal with writes), and so Google developed a number of specialized storage solutions[^13] that were enablers for most applications adopting a model where all state is transient.

不幸的是，大多數應用並不那麼簡單。可能會想到的一個自然而然問題是："這些持久的儲存解決方案是如何實現的—它們是*牛*嗎？" 答案應該是 "是的"。牛可以透過狀態複製來管理持久狀態。在不同的層面上，RAID陣列是一個類似的概念；我們將磁碟視為暫時的（接受其中一個可以消失的事實），同時仍保持主要狀態。在伺服器世界中，這可以透過多個副本來實現，多個副本儲存一個數據段並進行同步，以確保每個資料段都被複制足夠的次數（通常為3到5次）。請注意，正確設定此選項很困難（需要某種一致性處理方式來處理寫操作），因此Google開發了許多專門的儲存解決方案13，這些解決方案是採用所有狀態都是瞬態的模型的大多數應用程式的推動者。

Other types of local storage that cattle can use covers “re-creatable” data that is held locally to improve serving latency. Caching is the most obvious example here: a cache is nothing more than transient local storage that holds state in a transient location, but banks on the state not going away all the time, which allows for better performance characteristics on average. A key lesson for Google production infrastructure has been to provision the cache to meet your latency goals, but provision the core application for the total load. This has allowed us to avoid outages when the cache layer was lost because the noncached path was provisioned to handle the total load (although with higher latency). However, there is a clear trade-off here: how much to spend on the redundancy to mitigate the risk of an outage when cache capacity is lost.

牛可以使用的其他型別的本地儲存包括本地儲存的“可重新建立”資料，以改善服務延遲。快取是這裡最明顯的例子：快取只不過是在一個短暫的位置上儲存狀態的本地儲存，但卻依賴於該狀態不會一直消失，這使得平均效能特徵更好。谷歌生產基礎設施的一個關鍵經驗是，配置快取以滿足你的延遲要求，但為總負載配置核心應用程式。這使得我們能夠在快取層丟失時避免故障，因為非快取路徑的配置能夠處理總的負載（儘管延遲更高）。然而，這裡有一個明顯的權衡：當快取容量丟失時，要在冗餘上花多少錢才能減輕故障的風險。

In a similar vein to caching, data might be pulled in from external storage to local in the warm-up of an application, in order to improve request serving latency.

與快取類似，在應用程式的預熱過程中，資料可能從外部儲存拉到本地，以改善請求服務延遲。

One more case of using local storage—this time in case of data that’s written more than read—is batching writes. This is a common strategy for monitoring data (think, for instance, about gathering CPU utilization statistics from the fleet for the purposes of guiding the autoscaling system), but it can be used anywhere where it is acceptable for a fraction of data to perish, either because we do not need 100% data coverage (this is the monitoring case), or because the data that perishes can be re-created (this is the case of a batch job that processes data in chunks, and writes some output for each chunk). Note that in many cases, even if a particular calculation has to take a long time, it can be split into smaller time windows by periodic checkpointing of state to persistent storage.

還有一種使用本地儲存的情況--這次是在資料寫入多於讀取的情況下--是批量寫入。這是監控資料的常見策略（例如，考慮從機群中收集CPU利用率的統計資料，以指導自動伸縮系統），但它也可以用在任何可以接受部分資料丟失的地方，因為我們不需要100%的資料覆蓋（這是監控的情況），或者因為丟失的資料可以重新建立（這是一個批處理作業的情況，它分塊處理資料，並為每個分塊寫一些輸出）。請注意，在很多情況下，即使一個特定的計算需要很長的時間，也可以透過定期檢查狀態到永續性儲存的方式將其分割成更小的時間視窗。


> [^12]:	Note that, besides distributed state, there are other requirements to setting up an effective “servers as cattle” solution, like discovery and load-balancing systems (so that your application, which moves around the datacenter, can be accessed effectively). Because this book is less about building a full CaaS infrastructure and more about how such an infrastructure relates to the art of software engineering, we won’t go into more detail here./
> 12 請注意，除了分散式狀態，建立一個有效的 "伺服器即牛 "解決方案還有其他要求，比如發現和負載平衡系統（以便你的應用程式，在資料中心內移動，可以被有效訪問）。因為這本書與其說是關於建立一個完整的CaaS基礎設施，不如說是關於這樣的基礎設施與軟體工程藝術的關係，所以我們在這裡就不多說了。
> 
> [^13]:	See, for example, Sanjay Ghemawat, Howard Gobioff, and Shun-Tak Leung, “The Google File System,” Proceedings of the 19th ACM Symposium on Operating Systems, 2003; Fay Chang et al., “Bigtable: A Distributed Storage System for Structured Data,” 7th USENIX Symposium on Operating Systems Design and Implementation (OSDI); or James C. Corbett et al., “Spanner: Google’s Globally Distributed Database,” OSDI, 2012./
> 13 例如，見Sanjay Ghemawat, Howard Gobioff, and Shun-Tak Leung, "The Google File System," Pro- ceedings of the 19th ACM Symposium on Operating Systems, 2003; Fay Chang等人, "Bigtable: 一個結構化資料的分散式儲存系統，"第七屆USENIX作業系統設計和實施研討會（OSDI）；或James C. Corbett等人，”Spanner:谷歌的全球分散式資料庫"，OSDI，2012。


#### Connecting to a Service 連線到服務

As mentioned earlier, if anything in the system has the name of the host on which your program runs hardcoded (or even provided as a configuration parameter at startup), your program replicas are not cattle. However, to connect to your application, another application does need to get your address from somewhere. Where?

如前所述，如果系統中的任何內容都有你的程式所執行的主機的名字的硬編碼（甚至在啟動時作為配置引數提供），則程式副本不可用。然而，為了連線到你的應用程式，另一個應用程式確實需要從某個地方獲得你的地址。在哪裡？

The answer is to have an extra layer of indirection; that is, other applications refer to your application by some identifier that is durable across restarts of the specific “backend” instances. That identifier can be resolved by another system that the scheduler writes to when it places your application on a particular machine. Now, to avoid distributed storage lookups on the critical path of making a request to your application, clients will likely look up the address that your app can be found on, and set up a connection, at startup time, and monitor it in the background. This is generally called *service discovery*, and many compute offerings have built-in or modular solutions. Most such solutions also include some form of load balancing, which reduces coupling to specific backends even more.

答案是有一個額外的代理層；也就是說，其他應用程式透過某個識別符號來參考你的應用程式，這些識別符號在特定的 "後端 "實例的重啟中是持久的。這個識別符號可以由另一個系統來解決，當排程器把你的應用程式放在一個特定的機器上時，它就會寫到這個系統。現在，為了避免在向你的應用程式發出請求的關鍵路徑上進行分散式儲存查詢，客戶可能會在啟動時查詢你的應用程式的地址，並建立一個連線，並在後臺監控它。這通常被稱為*服務發現*，許多計算產品有內建或模組化的解決方案。大多數這樣的解決方案還包括某種形式的負載平衡，這就進一步減少了與特定後端的耦合。

A repercussion of this model is that you will likely need to repeat your requests in some cases, because the server you are talking to might be taken down before it manages to answer.[^14] Retrying requests is standard practice for network communication (e.g., mobile app to a server) because of network issues, but it might be less intuitive for things like a server communicating with its database. This makes it important to design the API of your servers in a way that handles such failures gracefully. For mutating requests, dealing with repeated requests is tricky. The property you want to guarantee is some variant of *idempotency—*that the result of issuing a request twice is the same as issuing it once. One useful tool to help with idempotency is client- assigned identifiers: if you are creating something (e.g., an order to deliver a pizza to a specific address), the order is assigned some identifier by the client; and if an order with that identifier was already recorded, the server assumes it’s a repeated request and reports success (it might also validate that the parameters of the order match).

這種模式的影響是，在某些情況下，你可能需要重複你的請求，因為你對話的伺服器可能在響應之前就被關閉了。由於網路問題，重試請求是網路通訊的標準做法（例如，移動應用程式到伺服器），但對於像伺服器與資料庫通訊的事情來說，這可能不夠直接。這使得在設計你的伺服器的API時，必須能夠優雅地處理這種故障。對於突變的請求，處理重複請求是很棘手的。你想保證的屬性是*冪等性變體*--發出一個請求兩次的結果與發出一次相同。幫助實現冪等性的一個有用工具是客戶機指定的識別符號：如果你正在建立一些東西（例如，將比薩餅送到一個特定的地址的訂單），該訂單由客戶端分配一些識別符號；如果一個具有該識別符號的訂單已經被記錄下來，伺服器會認為這是一個重複的請求並報告成功（它也可能驗證該訂單的引數是否匹配）。

One more surprising thing that we saw happen is that sometimes the scheduler loses contact with a particular machine due to some network problem. It then decides that all of the work there is lost and reschedules it onto other machines—and then the machine comes back! Now we have two programs on two different machines, both thinking they are “replica072.” The way for them to disambiguate is to check which one of them is referred to by the address resolution system (and the other one should terminate itself or be terminated); but it also is one more case for idempotency: two replicas performing the same work and serving the same role are another potential source of request duplication.

我們看到的另一件令人驚訝的事情是，有時排程器會因為一些網路問題而與某臺機器失去聯絡。然後它認為那裡的所有工作都丟失了，並將其重新安排到其他機器上--然後這臺機器又回來了! 現在我們在兩臺不同的機器上有兩個程式，都認為自己是 "replica072"。他們消除歧義的方法是檢查他們中的哪一個被地址解析系統提及（而另一個應該終止自己或被終止）；但這也是冪等性的另一個案例：兩個執行相同工作並擔任相同角色的副本是請求重複的另一個潛在來源。


> [^14]:	Note that retries need to be implemented correctly—with backoff, graceful degradation and tools to avoid cascading failures like jitter. Thus, this should likely be a part of Remote Procedure Call library, instead of implemented by hand by each developer. See, for example, Chapter 22: Addressing Cascading Failures in the SRE book./
> 14 請注意，重試需要正確地實現--用後退、優雅降級和工具來避免像抖動那樣的失敗。因此，這可能應該是遠端過程呼叫函式庫的一部分，而不是由每個開發人員手工實現。例如，見SRE書中的第22章：解決級聯故障。


### One-Off Code 一次性程式碼

Most of the previous discussion focused on production-quality jobs, either those serving user traffic, or data-processing pipelines producing production data. However, the life of a software engineer also involves running one-off analyses, exploratory prototypes, custom data-processing pipelines, and more. These need compute resources.

前面的討論大多集中在生產品質的工作上，要麼是那些為使用者流量服務的工作，要麼是產生生產資料的資料處理管道。然而，軟體工程師的生活也涉及到執行一次性分析、探索性原型、訂製資料處理管道等等。這些都需要計算資源。

Often, the engineer’s workstation is a satisfactory solution to the need for compute resources. If one wants to, say, automate the skimming through the 1 GB of logs that a service produced over the last day to check whether a suspicious line A always occurs before the error line B, they can just download the logs, write a short Python script, and let it run for a minute or two.

通常，工程師工作站是滿足計算資源需求的滿意解決方案。比如說，如果想自動瀏覽服務在最後一天產生的1GB日誌，以檢查可疑行a是否總是出現在錯誤行B之前，他們可以下載日誌，編寫一個簡短的Python指令碼，然後讓它執行一兩分鐘。

But if they want to automate the skimming through 1 TB of logs that service produced over the last year (for a similar purpose), waiting for roughly a day for the results to come in is likely not acceptable. A compute service that allows the engineer to just run the analysis on a distributed environment in several minutes (utilizing a few hundred cores) means the difference between having the analysis now and having it tomorrow. For tasks that require iteration—for example, if I will need to refine the query after seeing the results—the difference may be between having it done in a day and not having it done at all.

但是，如果他們想自動瀏覽去年服務生產的1 TB日誌（出於類似目的），等待大約一天的結果可能是不可接受的。一個允許工程師在幾分鐘內（利用幾百個核心）在分散式環境中執行分析的計算服務意味著現在進行分析和明天進行分析的區別。例如，對於需要迭代的任務，如果我在看到結果後需要優化查詢，那麼在一天內完成查詢和根本不完成查詢之間可能存在差異。

One concern that arises at times with this approach is that allowing engineers to just run one-off jobs on the distributed environment risks them wasting resources. This is, of course, a trade-off, but one that should be made consciously. It’s very unlikely that the cost of processing that the engineer runs is going to be more expensive than the engineer’s time spent on writing the processing code. The exact trade-off values differ depending on an organization’s compute environment and how much it pays its engineers, but it’s unlikely that a thousand core hours costs anything close to a day of engineering work. Compute resources, in that respect, are similar to markers, which we discussed in the opening of the book; there is a small savings opportunity for the company in instituting a process to acquire more compute resources, but this process is likely to cost much more in lost engineering opportunity and time than it saves.

這種方法有時會引起一個問題，即允許工程師在分散式環境中執行一次性作業可能會浪費資源。當然，這是一種權衡，但應該有意識地進行權衡。工程師執行的處理成本很可能不會比工程師寫處理程式碼的時間更貴。確切的權衡取決於一個組織的計算環境和它付給工程師的工資多少，但一千個核心小時的成本不太可能接近一天的工程工作。在這方面，計算資源類似於標記，我們在本書的開篇中討論過；對於公司來說，建立一個獲取更多計算資源的過程是一個很小的節約機會，但是這個過程在失去工程機會和時間方面的成本可能比它節省的成本高得多。

That said, compute resources differ from markers in that it’s easy to take way too many by accident. Although it’s unlikely someone will carry off a thousand markers, it’s totally possible someone will accidentally write a program that occupies a thousand machines without noticing.[^15] The natural solution to this is instituting quotas for resource usage by individual engineers. An alternative used by Google is to observe that because we’re running low-priority batch workloads effectively for free (see the section on multitenancy later on), we can provide engineers with almost unlimited quota for low-priority batch, which is good enough for most one-off engineering tasks.

這就是說，計算資源與標記的不同之處在於，很容易因意外而佔用過多的資源。雖然不太可能有人會攜帶上千個標記，但完全有可能有人會無意中編寫一個程式，在沒有注意到的情況下佔用了上千臺機器。解決這一問題的自然方法是為每個工程師的資源使用設定配額。谷歌使用的一個替代方案是，由於我們正在有效地免費執行低優先順序的批處理工作負載（見後面關於多租戶的部分），我們可以為工程師提供幾乎無限的低優先順序批處理配額，這對於大多數一次性工程任務來說已經足夠了。

> [^15]:	This has happened multiple times at Google; for instance, because of someone leaving load-testing infrastructure occupying a thousand Google Compute Engine VMs running when they went on vacation, or because a new employee was debugging a master binary on their workstation without realizing it was spawning 8,000 full-machine workers in the background./
> 15  這種情況在谷歌發生過多次；例如，因為有人在休假時留下了佔用一千臺谷歌計算引擎虛擬機器的負載測試基礎設施，或者因為一個新員工在他們的工作站上除錯一個主二進位制檔案時，沒有意識到它在後臺催生了8000個全機器worker。

## CaaS Over Time and Scale CaaS隨時間和規模的變化

We talked above how CaaS evolved at Google and the basic parts needed to make it happen—how the simple mission of “just give me resources to run my stuff ” translates to an actual architecture like Borg. Several aspects of how a CaaS architecture affects the life of software across time and scale deserve a closer look.

我們在上面討論了CaaS是如何在Google發展起來的，以及實現它所需要的基本部分--"只需給我資源來執行我的東西 "的簡單任務是如何過渡到像Borg這樣的架構。CaaS體系結構如何跨時間和規模影響軟體生命週期的幾個方面值得仔細研究。

### Containers as an Abstraction  容器是一種抽象

Containers, as we described them earlier, were shown primarily as an isolation mechanism, a way to enable multitenancy, while minimizing the interference between different tasks sharing a single machine. That was the initial motivation, at least in Google. But containers turned out to also serve a very important role in abstracting away the compute environment.

正如我們前面所描述的，容器主要是一種隔離機制，一種實現多租戶的方法，同時最大限度地減少共享一臺機器的不同任務之間的干擾。這是最初的動機，至少在谷歌是這樣。但事實證明，容器在抽象計算環境方面也起著非常重要的作用。

A container provides an abstraction boundary between the deployed software and the actual machine it’s running on. This means that as—over time—the machine changes, it is only the container software (presumably managed by a single team) that has to be adapted, whereas the application software (managed by each individual team, as the organization grows) can remain unchanged.

容器在部署的軟體和它所執行的實際機器之間提供了一個抽象邊界。這意味著，隨著時間的推移，機器發生了變化，只有容器軟體（大概由一個團隊管理）需要調整，而應用軟體（隨著組織的發展，由每個團隊管理）可以保持不變。

Let’s discuss two examples of how a containerized abstraction allows an organization to manage change.

讓我們來討論兩個例子，說明容器化的抽象如何讓一個組織管理變化。

A *filesystem abstraction* provides a way to incorporate software that was not written in the company without the need to manage custom machine configurations. This might be open source software an organization runs in its datacenter, or acquisitions that it wants to onboard onto its CaaS. Without a filesystem abstraction, onboarding a binary that expects a different filesystem layout (e.g., expecting a helper binary at */bin/foo/bar*) would require either modifying the base layout of all machines in the fleet, or fragmenting the fleet, or modifying the software (which might be difficult, or even impossible due to licence considerations).

*檔案系統抽象*提供了一種方法，可以將不是在公司編寫的軟體納入其中，而不需要管理自訂的機器配置。這可能是某個組織在其資料中心執行的開源軟體，或者是它想在其CaaS上進行的整合。在沒有檔案系統抽象的情況下，如果一個二進位制檔案需要一個不同的檔案系統佈局（例如，期望在*/bin/foo/bar*有一個附加二進位制檔案）將需要修改機群中所有機器的基本佈局，或者對叢集進行分段操作，或者修改軟體（這可能很困難，甚至由於許可證的考慮而不可能）。

Even though these solutions might be feasible if importing an external piece of software is something that happens once in a lifetime, it is not a sustainable solution if importing software becomes a common (or even only-somewhat-rare) practice.

即使這些解決方案可能是可行的，如果匯入一個外部軟體是一生中只會發生一次的事情，但如果匯入軟體成為一種常見的（甚至只是有點罕見的）做法，這不是一個可持續的解決方案。

A filesystem abstraction of some sort also helps with dependency management because it allows the software to predeclare and prepackage the dependencies (e.g., specific versions of libraries) that the software needs to run. Depending on the software installed on the machine presents a leaky abstraction that forces everybody to use the same version of precompiled libraries and makes upgrading any component very difficult, if not impossible.

某種型別的檔案系統抽象也有助於依賴性管理，因為它允許軟體預先宣告和預包裝軟體執行所需的依賴性（例如，特定版本的函式庫）。依賴於安裝在機器上的軟體提出了一個漏洞百出的抽象，迫使每個人都使用相同版本的預編譯函式庫，使任何元件的升級都非常困難，甚至不可能。

A container also provides a simple way to manage *named resources* on the machine. The canonical example is network ports; other named resources include specialized targets; for example, GPUs and other accelerators.

容器還提供了一種簡單的方法來管理計算機上的*命名資源*。典型的示例是網路埠；其他命名資源包括專用目標；例如GPU和其他加速器。

Google initially did not include network ports as a part of the container abstraction, and so binaries had to search for unused ports themselves. As a result, the PickUnu sedPortOrDie function has more than 20,000 usages in the Google C++ codebase. Docker, which was built after Linux namespaces were introduced, uses namespaces to provide containers with a virtual-private NIC, which means that applications can listen on any port they want. The Docker networking stack then maps a port on the machine to the in-container port. Kubernetes, which was originally built on top of Docker, goes one step further and requires the network implementation to treat containers (“pods” in Kubernetes parlance) as “real” IP addresses, available from the host network. Now every app can listen on any port they want without fear of conflicts.

Google最初並沒有將網路埠作為容器抽象的一部分，因此二進位制檔案不得不自己搜尋未使用的埠。結果，PickUnu sedPortOrDie函式在谷歌C++程式碼函式庫中有超過20,000次的使用。Docker是在Linux名稱空間引入後建立的，它使用名稱空間為容器提供虛擬私有網絡卡，這意味著應用程式可以監聽他們想要的任何埠。Docker網路堆疊然後將機器上的一個埠對映到容器內的埠。最初建立在Docker之上的Kubernetes更進一步，要求網路實現將容器（Kubernetes術語為 "pods"）視為 "真正的"IP地址，可從主機網路獲得。現在，每個應用程式都可以監聽他們想要的任何埠，而不用擔心衝突。

These improvements are particularly important when dealing with software not designed to run on the particular compute stack. Although many popular open source programs have configuration parameters for which port to use, there is no consistency between them for how to configure this.

當處理不是為在特定計算機技術棧上執行而設計的軟體時，這些改進尤其重要。儘管許多流行的開源程式都有使用哪個埠的配置引數，但它們之間對於如何配置並不一致。


#### Containers and implicit dependencies 容器和隱性依賴

As with any abstraction, Hyrum’s Law of implicit dependencies applies to the container abstraction. It probably applies *even more than usual*, both because of the huge number of users (at Google, all production software and much else will run on Borg) and because the users do not feel that they are using an API when using things like the filesystem (and are even less likely to think whether this API is stable, versioned, etc.).

與任何抽象一樣，海勒姆定律的隱性依賴適用於容器抽象。由於使用者數量巨大（在谷歌，所有生產軟體和許多其他軟體都將在Borg上執行），它可能比通常更適用而且因為使用者在使用檔案系統之類別的東西時感覺不到自己在使用API（而且更不可能考慮此API是否穩定、是否有版本等）。

To illustrate, let’s return to the example of process ID space exhaustion that Borg experienced in 2011. You might wonder why the process IDs are exhaustible. Are they not simply integer IDs that can be assigned from the 32-bit or 64-bit space? In Linux, they are in practice assigned in the range [0,..., PID_MAX - 1], where PID_MAX defaults to 32,000. PID_MAX, however, can be raised through a simple configuration change (to a considerably higher limit). Problem solved?

為了說明這一點，讓我們回到Borg在2011年經歷的程序ID空間耗盡的例子。你可能想知道為什麼程序ID是可耗盡的。它們不僅僅是可以從32位或64位空間分配的整數ID嗎？在Linux中，它們實際上是在[0，…，PID_MAX-1]範圍內分配的，其中PID_MAX預設為32000。然而，PID_MAX可以透過簡單的配置更改（達到相當高的限制）來提高。問題解決了嗎？

Well, no. By Hyrum’s Law, the fact that the PIDs that processes running on Borg got were limited to the 0...32,000 range became an implicit API guarantee that people started depending on; for instance, log storage processes depended on the fact that the PID can be stored in five digits, and broke for six-digit PIDs, because record names exceeded the maximum allowed length. Dealing with the problem became a lengthy, two-phase project. First, a temporary upper bound on the number of PIDs a single container can use (so that a single thread-leaking job cannot render the whole machine unusable). Second, splitting the PID space for threads and processes. (Because it turned out very few users depended on the 32,000 guarantee for the PIDs assigned to threads, as opposed to processes. So, we could increase the limit for threads and keep it at 32,000 for processes.) Phase three would be to introduce PID namespaces to Borg, giving each container its own complete PID space. Predictably (Hyrum’s Law again), a multitude of systems ended up assuming that the triple {hostname, timestamp, pid} uniquely identifies a process, which would break if PID namespaces were introduced. The effort to identify all these places and fix them (and backport any relevant data) is still ongoing eight years later.

嗯，沒有。根據海勒姆定律，在Borg上執行的程序得到的PID被限制在0...32,000範圍內，這一事實成為人們開始依賴的隱含的API保證；例如，日誌儲存程序依賴於PID可以儲存為五位數的事實，而對於六位數的PID來說，就會出現問題，因為記錄名稱超出了最大允許長度。處理這個問題成為一個漫長的、分兩個階段的專案。首先，對單個容器可以使用的PID數量設定一個臨時的上限（這樣單個執行緒洩漏的工作就不會導致整個機器無法使用）。第二，為執行緒和程序分割PID空間。(因為事實證明，很少有使用者依賴分配給執行緒的PID的32000個保證，而不是程序。所以，我們可以增加執行緒的限制，而將程序的限制保持在32,000個）。第三階段是在Borg中引入PID名稱空間，讓每個容器擁有自己完整的PID空間。可以預見的是（又是Hyrum定律），許多系統最終都認為{hostname, timestamp, pid}這三者可以唯一地識別一個程序，如果引入PID名稱空間，這將會被打破。識別所有這些地方並修復它們（以及回傳任何相關資料）的努力在八年後仍在進行。

The point here is not that you should run your containers in PID namespaces. Although it’s a good idea, it’s not the interesting lesson here. When Borg’s containers were built, PID namespaces did not exist; and even if they did, it’s unreasonable to expect engineers designing Borg in 2003 to recognize the value of introducing them. Even now there are certainly resources on a machine that are not sufficiently isolated, which will probably cause problems one day. This underlines the challenges of designing a container system that will prove maintainable over time and thus the value of using a container system developed and used by a broader community, where these types of issues have already occurred for others and the lessons learned have been incorporated.

這裡的重點不是說你應該在PID名稱空間中執行你的容器。儘管這是個好主意，但這並不是有趣的經驗。當Borg的容器被建立時，PID名稱空間並不存在；即使它們存在，期望2003年設計Borg的工程師認識到引入它們的價值也是不合理的。即使是現在，機器上也肯定有一些資源沒有被充分隔離，這可能會在某一天造成問題。這強調了設計一個容器系統的挑戰，該系統將被證明是可以長期維護的，因此使用一個由更廣泛的社群開發和使用的容器系統的價值，在那裡，這些型別的問題已經存在為其他人發生的事件，已將所吸取的經驗教訓納入其中。

### One Service to Rule Them All 一種服務統治一切

As discussed earlier, the original WorkQueue design was targeted at only some batch jobs, which ended up all sharing a pool of machines managed by the WorkQueue, and a different architecture was used for serving jobs, with each particular serving job running in its own, dedicated pool of machines. The open source equivalent would be running a separate Kubernetes cluster for each type of workload (plus one pool for all the batch jobs).

如前所述，最初的WorkQueue設計只針對一些批處理作業，這些作業最終都共享一個由WorkQueue管理的機器資源池，而對於服務作業則採用不同的架構，每個特定的服務作業都執行在自己的專用機器資源池中。開放原始碼的做法是為每種工作負載執行一個單獨的Kubernetes叢集（加上一個用於所有批處理工作的池）。

In 2003, the Borg project was started, aiming (and eventually succeeding at) building a compute service that assimilates these disparate pools into one large pool. Borg’s pool covered both serving and batch jobs and became the only pool in any datacenter (the equivalent would be running a single large Kubernetes cluster for all workloads in each geographical location). There are two significant efficiency gains here worth discussing.

2003年，Borg專案啟動，旨在（並最終成功地）建立一個計算服務，將這些不同的機器資源池整合為一個大機器資源池。Borg的機器資源池涵蓋了服務和批處理工作，併成為任何資料中心的唯一機器資源池（相當於為每個地理位置的所有工作負載執行一個大型Kubernetes叢集）。這裡有兩個顯著的效率提升值得討論。

The first one is that serving machines became cattle (the way the Borg design doc put it: “*Machines are anonymous:* programs don’t care which machine they run on as long as it has the right characteristics”). If every team managing a serving job must manage their own pool of machines (their own cluster), the same organizational overhead of maintaining and administering that pool is applied to every one of these teams. As time passes, the management practices of these pools will diverge over time, making company-wide changes (like moving to a new server architecture, or switching datacenters) more and more complex. A unified management infrastructure—that is, a *common* compute service for all the workloads in the organization—allows Google to avoid this linear scaling factor; there aren’t *n* different management practices for the physical machines in the fleet, there’s just Borg.[^16]

第一個是，服務於機器的人變成了牛（Borg設計文件是這樣說的。"*機器是透明的：*程式並不關心它們在哪臺機器上執行，只要它有正確的特徵"）。如果每個管理服務工作的團隊都必須管理他們自己的機器資源池（他們自己的叢集），那麼維護和管理這個機器資源池的組織開銷也同樣適用於這些團隊中的每個人。隨著時間的推移，這些機器資源池的管理實踐會隨著時間的推移而產生分歧，使整個公司範圍內的變化（如轉移到一個新的伺服器架構，或切換資料中心）變得越來越複雜。一個統一的管理基礎設施--也就是一個適用於組織中所有工作負載的*通用*計算服務--允許谷歌避免這種線性擴充套件因素；對於機群中的物理機器沒有*N*種不同的管理實踐，只有Borg。

The second one is more subtle and might not be applicable to every organization, but it was very relevant to Google. The distinct needs of batch and serving jobs turn out to be complementary. Serving jobs usually need to be overprovisioned because they need to have capacity to serve user traffic without significant latency decreases, even in the case of a usage spike or partial infrastructure outage. This means that a machine running only serving jobs will be underutilized. It’s tempting to try to take advantage of that slack by overcommitting the machine, but that defeats the purpose of the slack in the first place, because if the spike/outage does happen, the resources we need will not be available.

第二個問題更加微妙，可能並不適用於每個組織，但它與谷歌非常相關。批量作業和服務作業的不同需求是互補的。服務工作通常需要超額配置，因為它們需要有能力為使用者流量提供服務而不出現明顯的延遲下降，即使在使用量激增或部分基礎設施中斷的情況下。這意味著僅執行服務作業的機器將未得到充分利用。試圖透過過度使用機器來利用這種閒置是很有誘惑力的，但這首先違背了閒置的目的，因為如果出現高峰/中斷出現，我們需要的資源將無法使用。

However, this reasoning applies only to serving jobs! If we have a number of serving jobs on a machine and these jobs are requesting RAM and CPU that sum up to the total size of the machine, no more serving jobs can be put in there, even if real utilization of resources is only 30% of capacity. But we *can* (and, in Borg, will) put batch jobs in the spare 70%, with the policy that if any of the serving jobs need the memory or CPU, we will reclaim it from the batch jobs (by freezing them in the case of CPU or killing in the case of RAM). Because the batch jobs are interested in throughput (measured in aggregate across hundreds of workers, not for individual tasks) and their individual replicas are cattle anyway, they will be more than happy to soak up this spare capacity of serving jobs.

然而，這種推理僅適用於服務作業！如果我們在一臺機器上有許多服務作業，而這些作業請求的RAM和CPU總計為機器的總和，即使資源的實際利用率僅為容量的30%，也不能在其中放置更多的服務作業。但我們可以（而且，在Borg，我們）將批處理作業放在備用70%中，策略是，如果任何服務作業需要記憶體或CPU，我們將從批處理作業中回收（在CPU的情況下凍結它們，在RAM的情況下殺死它們）。因為批處理作業對吞吐量感興趣（在數百名worker中進行聚合測量，而不是針對單個任務），而且它們的單個副本無論如何都是牛，所以它們將非常樂意吸收服務作業的這一剩餘容量。

Depending on the shape of the workloads in a given pool of machines, this means that either all of the batch workload is effectively running on free resources (because we are paying for them in the slack of serving jobs anyway) or all the serving workload is effectively paying for only what they use, not for the slack capacity they need for failure resistance (because the batch jobs are running in that slack). In Google’s case, most of the time, it turns out we run batch effectively for free.

根據給定機器資源池池中工作負載的形狀，這意味著要麼所有批處理工作負載都有效地執行在空閒資源上（因為我們無論如何都是在空閒的服務作業中為它們付費）或者，所有的服務性工作負載實際上只為他們使用的東西付費，而不是為他們抵抗故障所需的閒置容量付費（因為批處理作業是在這種閒置狀態下執行的）。在谷歌的案例中，大多數時候，事實證明我們免費有效地執行批處理。

>[^16]:	As in any complex system, there are exceptions. Not all machines owned by Google are Borg-managed, and not every datacenter is covered by a single Borg cell. But the majority of engineers work in an environment in which they don’t touch non-Borg machines, or nonstandard cells./
> 16 正如任何複雜的系統一樣，也有例外。並非所有谷歌擁有的機器都由Borg管理，也不是每個資料中心都由一個Borg單元覆蓋。但大多數工程師的工作環境是，他們不接觸非Borg機，也不接觸非標準的單元。


#### Multitenancy for serving jobs 為工作提供服務的多租戶

Earlier, we discussed a number of requirements that a compute service must satisfy to be suitable for running serving jobs. As previously discussed, there are multiple advantages to having the serving jobs be managed by a common compute solution, but this also comes with challenges. One particular requirement worth repeating is a discovery service, discussed in [“Connecting to a Service” on page 528](#_bookmark2176). There are a number of other requirements that are new when we want to extend the scope of a managed compute solution to serving tasks, for example:

•   Rescheduling of jobs needs to be throttled: although it’s probably acceptable to kill and restart 50% of a batch job’s replicas (because it will cause only a temporary blip in processing, and what we really care about is throughput), it’s unlikely to be acceptable to kill and restart 50% of a serving job’s replicas (because the remaining jobs are likely too few to be able to serve user traffic while waiting for the restarted jobs to come back up again).

•   A batch job can usually be killed without warning. What we lose is some of the already performed processing, which can be redone. When a serving job is killed without warning, we likely risk some user-facing traffic returning errors or (at best) having increased latency; it is preferable to give several seconds of warning ahead of time so that the job can finish serving requests it has in flight and not accept new ones.

早些時候，我們討論了計算服務必須滿足的一些要求，以適合執行服務作業。正如之前所討論的，讓服務作業由一個共同的計算解決方案來管理有多種好處，但這也伴隨著挑戰。一個值得重複的特殊要求是發現服務，在[第528頁的 "連線到服務"]中討論過。當我們想把託管計算解決方案的範圍擴充套件到服務任務時，還有一些其他的要求是新的，比如說。

- 作業的重新排程需要節制：儘管殺死並重新啟動一個批處理作業的50%的副本可能是可以接受的（因為這隻會導致處理過程中的暫時性突變，而我們真正關心的是吞吐量），但殺死並重新啟動一個服務作業的50%的副本是不太可能接受的（因為剩下的作業可能太少，無法在等待重新啟動的作業再次出現的同時為使用者流量提供服務）。

- 一個批處理作業通常可以在沒有警告的情況下被殺死。我們失去的是一些已經執行的處理，這些處理可以重新進行。當一個服務工作在沒有警告的情況下被殺死時，我們很可能冒著一些面向使用者的流量返回錯誤或（最多）延遲增加的風險；最好是提前幾秒鐘發出警告，以便工作能夠完成服務它在執行中的請求，不再接受新的請求。

For the aforementioned efficiency reasons, Borg covers both batch and serving jobs, but multiple compute offerings split the two concepts—typically, a shared pool of machines for batch jobs, and dedicated, stable pools of machines for serving jobs. Regardless of whether the same compute architecture is used for both types of jobs, however, both groups benefit from being treated like cattle.

出於上述效率原因，Borg同時涵蓋了批處理和服務作業，但多個計算產品將這兩個概念分割開來--通常情況下，批處理作業使用共享的機器資源池，而服務工作使用專用的、穩定的機器資源池。然而，無論這兩類別工作是否使用相同的計算架構，這兩類別工作都會因被當作牛一樣對待而受益。

### Submitted Configuration 提交配置

The Borg scheduler receives the configuration of a replicated service or batch job to run in the cell as the contents of a Remote Procedure Call (RPC). It’s possible for the operator of the service to manage it by using a command-line interface (CLI) that sends those RPCs, and have the parameters to the CLI stored in shared documentation, or in their head.

Borg排程器接收擴容服務或批處理作業的配置，作為遠端過程呼叫（RPC）的內容在單元中執行。服務運營商可以透過使用命令列介面（CLI）對其進行管理，該介面傳送這些RPC，並將引數儲存在共享文件或其Header中。

Depending on documentation and tribal knowledge over code submitted to a repository is rarely a good idea in general because both documentation and tribal knowledge have a tendency to deteriorate over time (see [Chapter 3](#_bookmark182)). However, the next natural step in the evolution—wrapping the execution of the CLI in a locally developed script—is still inferior to using a dedicated configuration language to specify the configuration of your service.

在一般情況下，依靠文件和團隊知識而不是提交給資源函式庫的程式碼不會是個好主意，因為文件和團隊知識都有隨著時間推移而退化的趨勢（見[第三章]）。然而，前進中的下一個自然步驟--將CLI的執行包裹在本地開發的指令碼中--仍然不如使用專門的配置語言來指定服務的配置。

Over time, the runtime presence of a logical service will typically grow beyond a single set of replicated containers in one datacenter across many axes:
- It will spread its presence across multiple datacenters (both for user affinity and failure resistance).
- It will fork into having staging and development environments in addition to the production environment/configuration.
- It will accrue additional replicated containers of different types in the form of attached services, like a memcached accompanying the service.

隨著時間的推移，邏輯服務的執行時存在通常會超過在一個數據中心的部署容器組，跨越多個區域：
- 它將在多個數據中心分散其存在（既有使用者親和力，也有抗故障能力）。
- 除了生產環境/配置之外，它還會分叉到擁有臨時和開發環境。
- 它將以附加服務的形式累積不同型別的額外副本容器，如服務附帶的memcached。

Management of the service is much simplified if this complex setup can be expressed in a standardized configuration language that allows easy expression of standard operations (like “update my service to the new version of the binary, but taking down no more than 5% of capacity at any given time”).

如果這種複雜的設定可以用一種標準化的配置語言來表達，那麼服務的管理就會大大簡化，這種語言可以方便地表達標準操作（比如“將我的服務更新為新版本的二進位制檔案，但在任何給定時間佔用的容量不超過5%”）。

A standardized configuration language provides standard configuration that other teams can easily include in their service definition. As usual, we emphasize the value of such standard configuration over time and scale. If every team writes a different snippet of custom code to stand up their memcached service, it becomes very difficult to perform organization-wide tasks like swapping out to a new memcache implementation (e.g., for performance or licencing reasons) or to push a security update to all the memcache deployments. Also note that such a standardized configuration language is a requirement for automation in deployment (see [Chapter 24](#_bookmark2100)).

標準化配置語言提供標準配置，其他團隊可以輕鬆地將其包含在服務定義中。像往常一樣，我們強調這種標準配置在時間和規模上的價值。如果每個團隊都編寫不同的自訂程式碼片段以支援其memcache服務，則執行組織範圍內的任務（如切換到新的memcache實現）或將安全更新推送到所有memcache部署將變得非常困難。還要注意，這種標準化配置語言是部署自動化的一個要求（參見第24章

## Choosing a Compute Service 選擇計算服務

It’s unlikely any organization will go down the path that Google went, building its own compute architecture from scratch. These days, modern compute offerings are available both in the open source world (like Kubernetes or Mesos, or, at a different level of abstraction, OpenWhisk or Knative), or as public cloud managed offerings (again, at different levels of complexity, from things like Google Cloud Platform’s Managed Instance Groups or Amazon Web Services Elastic Compute Cloud [Amazon EC2] autoscaling; to managed containers similar to Borg, like Microsoft Azure Kubernetes Service [AKS] or Google Kubernetes Engine [GKE]; to a serverless offering like AWS Lambda or Google’s Cloud Functions).

不太可能有別的組織會重走谷歌走過的路，從頭開始建構自己的計算架構。如今，現代計算產品在開源世界（比如Kubernetes或Mesos，或者在不同的抽象層次上，OpenWhisk或Knative），或作為公共雲管理產品（同樣，在不同的複雜性級別，從Google雲平臺的託管實例組或Amazon Web服務彈性計算雲[Amazon EC2]自動伸縮；到類似於Borg的託管容器，如Microsoft Azure Kubernetes服務[AKS]或谷歌Kubernetes引擎[GKE]；提供類似AWS Lambda或谷歌雲功能的無伺服器服務）。

However, most organizations will *choose* a compute service, just as Google did internally. Note that a compute infrastructure has a high lock-in factor. One reason for that is because code will be written in a way that takes advantage of all the properties of the system (Hyrum’s Law); thus, for instance, if you choose a VM-based offering, teams will tweak their particular VM images; and if you choose a specific container- based solution, teams will call out to the APIs of the cluster manager. If your architecture allows code to treat VMs (or containers) as pets, teams will do so, and then a move to a solution that depends on them being treated like cattle (or even different forms of pets) will be difficult.

然而，大多陣列織會*選擇一個計算服務*，就像谷歌內部那樣。請注意，計算基礎設施有一個很高的鎖定因素。其中一個原因是，程式碼的編寫方式將充分利用系統的所有特性（海勒姆定律）；因此，例如，如果你選擇了一個基於虛擬機器的產品，團隊將調整他們特定的虛擬機器映象；如果你選擇了一個特定的基於容器的解決方案，團隊將呼叫叢集管理器的API。如果您的架構允許程式碼將虛擬機器（或容器）視為寵物，那麼團隊將這樣做，然後轉向一種解決方案，將它們視為牛（甚至不同形式的寵物）將是困難的。

To show how even the smallest details of a compute solution can end up locked in, consider how Borg runs the command that the user provided in the configuration. In most cases, the command will be the execution of a binary (possibly followed by a number of arguments). However, for convenience, the authors of Borg also included the possibility of passing in a shell script; for example, while true; do ./ my_binary; done.[^17] However, whereas a binary execution can be done through a simple fork-and-exec (which is what Borg does), the shell script needs to be run by a shell like Bash. So, Borg actually executed /usr/bin/bash -c $USER_COMMAND, which works in the case of a simple binary execution as well.

為了說明即使是計算解決方案中最小的細節也會最終被鎖定，考慮一下Borg如何執行使用者在配置中提供的命令。在大多數情況下，該命令將是執行一個二進位制檔案（後面可能有一些引數）。然而，為了方便起見，Borg的作者也包括了傳入一個shell指令碼的可能性；例如，`while true; do ./ my_binary; done`。 然而，二進位制的執行可以透過一個簡單的fork-and-exec來完成（這就是Borg的做法），shell指令碼需要由一個像Bash這樣的shell來執行。所以，Borg實際上是執行/usr/bin/bash -c $USER_COMMAND，該命令也適用於簡單的二進位制執行。

At some point, the Borg team realized that at Google’s scale, the resources—mostly memory—consumed by this Bash wrapper are non-negligible, and decided to move over to using a more lightweight shell: ash. So, the team made a change to the process runner code to run /usr/bin/ash -c $USER_COMMAND instead.

在某種程度上，Borg團隊意識到在Google的規模下，這個Bash包裝器所消耗的資源--主要是記憶體--是不可忽視的，並決定轉而使用一個更輕量級的shell：ash。因此，該團隊對程序執行器的程式碼進行了修改，改為執行`/usr/bin/ash -c $USER_COMMAND`。

You would think that this is not a risky change; after all, we control the environment, we know that both of these binaries exist, and so there should be no way this doesn’t work. In reality, the way this didn’t work is that the Borg engineers were not the first to notice the extra memory overhead of running Bash. Some teams were creative in their desire to limit memory usage and replaced (in their custom filesystem overlay) the Bash command with a custom-written piece of “execute the second argument” code. These teams, of course, were very aware of their memory usage, and so when the Borg team changed the process runner to use ash (which was not overwritten by the custom code), their memory usage increased (because it started including ash usage instead of the custom code usage), and this caused alerts, rolling back the change, and a certain amount of unhappiness.

你會認為這不是一個有風險的改變；畢竟，我們控制了環境，我們知道這兩個二進位制檔案都存在，所以這不可能不起作用。事實上，這不起作用的原因是，Borg的工程師們並不是第一個注意到執行Bash的額外記憶體開銷的人。一些團隊在限制記憶體使用方面很有創意，他們（在他們的自訂檔案系統覆蓋中）用一段自訂編寫的 "執行第二個引數 "的程式碼來替換Bash命令。當然，這些團隊非常清楚他們的記憶體使用情況，因此當Borg團隊將程序執行器改為使用ash（沒有被自訂程式碼覆蓋）時，他們的記憶體使用量增加了（因為它開始包括ash的nei使用量而不是自訂程式碼的記憶體使用量），這引起了警報、回滾變化和一定程度的不愉快。

Another reason that a compute service choice is difficult to change over time is that any compute service choice will eventually become surrounded by a large ecosystem of helper services—tools for logging, monitoring, debugging, alerting, visualization, on-the-fly analysis, configuration languages and meta-languages, user interfaces, and more. These tools would need to be rewritten as a part of a compute service change, and even understanding and enumerating those tools is likely to be a challenge for a medium or large organization.

計算服務的選擇難以隨時間變化的另一個原因是，任何計算服務的選擇最終都會被一個龐大的輔助服務生態系統所包圍--用於記錄、監控、除錯、警報、視覺化、即時分析、配置語言和元語言、使用者介面等等的工具。這些工具需要作為計算服務變革的一部分被重寫，甚至理解和列舉這些工具對於一個大中型組織來說都可能是一個挑戰。

Thus, the choice of a compute architecture is important. As with most software engineering choices, this one involves trade-offs. Let’s discuss a few.

因此，計算架構的選擇是很重要的。與大多數軟體工程的選擇一樣，這個選擇涉及到權衡。讓我們來討論一下。

> [^17]:	This particular command is actively harmful under Borg because it prevents Borg’s mechanisms for dealing with failure from kicking in. However, more complex wrappers that echo parts of the environment to logging, for example, are still in use to help debug startup problems. /
> 17  這個特殊的命令在Borg下是有害的，因為它阻止Borg處理故障的機制啟動。但是，更復雜的包裝器（例如，將環境的一部分回送到日誌記錄）仍然在使用，以幫助除錯啟動問題。


### Centralization Versus Customization 統一與訂製

From the point of view of management overhead of the compute stack (and also from the point of view of resource efficiency), the best an organization can do is adopt a single CaaS solution to manage its entire fleet of machines and use only the tools available there for everybody. This ensures that as the organization grows, the cost of managing the fleet remains manageable. This path is basically what Google has done with Borg.

從計算棧的管理開銷的角度來看（也從資源效率的角度來看），一個組織能做的最好的事情就是統一採用一個的CaaS解決方案來管理它的整個機群，並且只使用那裡的工具供大家使用。這可以確保隨著組織的發展，管理叢集的成本仍然是可控的。這條路基本上就是谷歌對Borg所做的。

#### Need for customization 訂製化

However, a growing organization will have increasingly diverse needs. For instance, when Google launched the Google Compute Engine (the “VM as a Service” public cloud offering) in 2012, the VMs, just as most everything else at Google, were managed by Borg. This means that each VM was running in a separate container controlled by Borg. However, the “cattle” approach to task management did not suit Cloud’s workloads, because each particular container was actually a VM that some particular user was running, and Cloud’s users did not, typically, treat the VMs as cattle.[^18]

然而，一個不斷髮展的組織將有越來越多樣化的需求。例如，當谷歌在2012年推出谷歌計算引擎（“虛擬機器即服務”公共雲產品）時，這些虛擬機器與谷歌的大多數其他產品一樣，都是Borg設計的。這意味著每個虛擬機器都在博格控制的單獨容器中執行。然而，任務管理的“牛”方法並不適合雲的工作負載，因為每個特定容器實際上是某個特定使用者正在執行的VM，而云的使用者通常不會將VM視為牛。

Reconciling this difference required considerable work on both sides. The Cloud organization made sure to support live migration of VMs; that is, the ability to take a VM running on one machine, spin up a copy of that VM on another machine, bring the copy to be a perfect image, and finally redirect all traffic to the copy, without causing a noticeable period when service is unavailable.[^19] Borg, on the other hand, had to be adapted to avoid at-will killing of containers containing VMs (to provide the time to migrate the VM’s contents to the new machine), and also, given that the whole migration process is more expensive, Borg’s scheduling algorithms were adapted to optimize for decreasing the risk of rescheduling being needed.[^20] Of course, these modifications were rolled out only for the machines running the cloud workloads, leading to a (small, but still noticeable) bifurcation of Google’s internal compute offering.

調和這種差異需要雙方做大量的工作。雲端計算組織確保支援虛擬機器的即時遷移；也就是說，能夠在一臺機器上執行一個虛擬機器，在另一臺機器上啟動該虛擬機器的副本，使該副本成為一個完美的映象，並最終將所有流量重新導向到該副本，而不會造成明顯的服務不可用期。  另一方面，Borg必須進行調整，以避免隨意殺死包含虛擬機器的容器（以提供時間將虛擬機器的內容遷移到新機器上），同時，鑑於整個遷移過程更加耗時，Borg的排程演算法被調整為優化，以減少需要重新排程的風險。當然，這些修改只針對運行雲工作負載的機器，導致了谷歌內部計算產品的分化（很小，但仍然很明顯）。

> [^18]:	My mail server is not interchangeable with your graphics rendering job, even if both of those tasks are running in the same form of VM.
> 18  我的郵件伺服器不能與圖形渲染作業互換，即使這兩個任務都以相同的VM形式執行。
>
> [^19]:	This is not the only motivation for making user VMs possible to live migrate; it also offers considerable user- facing benefits because it means the host operating system can be patched and the host hardware updated without disrupting the VM. The alternative (used by other major cloud vendors) is to deliver “maintenance event notices,” which mean the VM can be, for example, rebooted or stopped and later started up by the cloud provider./
> 19  這不是讓使用者虛擬機器能夠即時遷移的唯一動機；它還提供了大量面向使用者的好處，因為這意味著可以修補主機作業系統並更新主機硬體，而不會中斷VM。另一種選擇（其他主要雲供應商使用）是提供“維護事件通知”，這意味著雲提供商可以重新啟動或停止VM，然後再啟動VM。
>
> [^20]: This is particularly relevant given that not all customer VMs are opted into live migration; for some workloads even the short period of degraded performance during the migration is unacceptable. These customers will receive maintenance event notices, and Borg will avoid evicting the containers with those VMs unless strictly necessary./
> 20  考慮到並非所有客戶虛擬機器都選擇即時遷移，這一點尤其重要；對於某些工作負載，即使在遷移過程中出現短期效能下降也是不可接受的。這些客戶將收到維護事件通知，除非嚴格必要，否則Borg將避免驅逐帶有這些VM的容器。

A different example—but one that also leads to a bifurcation—comes from Search. Around 2011, one of the replicated containers serving Google Search web traffic had a giant index built up on local disks, storing the less-often-accessed part of the Google index of the web (the more common queries were served by in-memory caches from other containers). Building up this index on a particular machine required the capacity of multiple hard drives and took several hours to fill in the data. However, at the time, Borg assumed that if any of the disks that a particular container had data on had gone bad, the container will be unable to continue, and needs to be rescheduled to a different machine. This combination (along with the relatively high failure rate of spinning disks, compared to other hardware) caused severe availability problems; containers were taken down all the time and then took forever to start up again. To address this, Borg had to add the capability for a container to deal with disk failure by itself, opting out of Borg’s default treatment; while the Search team had to adapt the process to continue operation with partial data loss.

一個不同的例子--但也導致了分叉--來自於搜尋。2011年左右，一個為谷歌搜尋網路流量服務的複製容器在本地磁碟上建立了一個巨大的索引，儲存了谷歌網路索引中不常被訪問的部分（更常見的查詢由其他容器的記憶體快取提供）。在一臺特定的機器上建立這個索引需要多個硬碟的容量，並且需要幾個小時來填入資料。然而，在當時，Borg認為，如果某個特定容器上有資料的任何磁碟壞了，該容器將無法繼續執行，需要重新排程到另一臺機器上。這種組合（與其他硬體相比，旋轉磁碟的故障率相對較高）造成了嚴重的可用性問題；容器總是被關閉，然後又要花很長時間才能重新啟動。為了解決這個問題，Borg必須增加容器自己處理磁碟故障的能力，選擇不使用Borg的預設處理方式；而搜尋團隊必須調整流程，在部分資料丟失的情況下繼續執行。

Multiple other bifurcations, covering areas like filesystem shape, filesystem access, memory control, allocation and access, CPU/memory locality, special hardware, special scheduling constraints, and more, caused the API surface of Borg to become large and unwieldy, and the intersection of behaviors became difficult to predict, and even more difficult to test. Nobody really knew whether the expected thing happened if a container requested *both* the special Cloud treatment for eviction *and* the custom Search treatment for disk failure (and in many cases, it was not even obvious what “expected” means).

其他多個分叉，涵蓋了檔案系統形狀、檔案系統訪問、記憶體控制、分配和訪問、CPU/記憶體定位、特殊硬體、特殊排程約束等領域，導致Borg的API體量變得龐大而笨重，各種行為的交叉點變得難以預測，甚至更難測試。沒有人真正知道，如果一個容器同時請求特殊的雲處理（用於驅逐）和自訂的磁碟故障搜尋處理（在許多情況下，“預期”的含義甚至不明顯），預期的事情是否會發生。


After 2012, the Borg team devoted significant time to cleaning up the API of Borg. It discovered some of the functionalities Borg offered were no longer used at all.[^21] The more concerning group of functionalities were those that were used by multiple containers, but it was unclear whether intentionally—the process of copying the configuration files between projects led to proliferation of usage of features that were originally intended for power users only. Whitelisting was introduced for certain features to limit their spread and clearly mark them as poweruser–only. However, the cleanup is still ongoing, and some changes (like using labels for identifying groups of containers) are still not fully done.[^22]

2012年後，Borg團隊花了大量時間來清理Borg的API。它發現博格提供的一些功能已不再使用。令人關注的功能組是多個容器使用的功能組，但目前尚不清楚，在專案之間複製配置檔案的過程是否有意導致原本只針對超級使用者的功能的使用激增。某些功能被引入了白名單，以限制它們的傳播，並明確地將它們標記為僅適用於特權使用者。然而，清理工作仍在進行，一些變化（如使用標籤來識別容器組）仍未完全完成。

As usual with trade-offs, although there are ways to invest effort and get some of the benefits of customization while not suffering the worst downsides (like the aforementioned whitelisting for power functionality), in the end there are hard choices to be made. These choices usually take the form of multiple small questions: do we accept expanding the explicit (or worse, implicit) API surface to accommodate a particular user of our infrastructure, or do we significantly inconvenience that user, but maintain higher coherence?

與通常的權衡方法一樣，儘管有一些方法可以投入精力並從訂製中獲得一些好處，同時又不會遭受最壞的負面影響（如前面提到的特權白名單），但最終還是要做出艱難的選擇。這些選擇通常以多個小問題的形式出現：我們是否接受擴充套件顯式（或更糟的是，隱式）API表面以適應我們基礎設施的特定使用者，或者我們是否顯著地給該使用者帶來不便，但主要是保持更高的一致性？

> [^21]:	A good reminder that monitoring and tracking the usage of your features is valuable over time./
> 21  這是一個很好的提醒，隨著時間的推移，監視和追蹤功能的使用是很有價值的。
> 
> [^22]:	This means that Kubernetes, which benefited from the experience of cleaning up Borg but was not hampered by a broad existing userbase to begin with, was significantly more modern in quite a few aspects (like its treatment of labels) from the beginning. That said, Kubernetes suffers some of the same issues now that it has broad adoption across a variety of types of applications./
> 22  這意味著Kubernetes從清理Borg的經驗中獲益，但從一開始就沒有受到廣泛的現有使用者基礎的阻礙，從一開始就在很多方面（如標籤的處理）明顯更加現代化。也就是說，Kubernetes現在也遇到了一些相同的問題，因為它在各種型別的應用程式中得到了廣泛的採用。

### Level of Abstraction: Serverless 抽象級別：無伺服器

The description of taming the compute environment by Google can easily be read as a tale of increasing and improving abstraction—the more advanced versions of Borg took care of more management responsibilities and isolated the container more from the underlying environment. It’s easy to get the impression this is a simple story: more abstraction is good; less abstraction is bad.

谷歌對馴服計算環境的描述很容易被理解為一個增加和改進抽象的故事--更進階的Borg版本承擔了更多的管理責任，並將容器與底層環境更多地隔離。這很容易讓人覺得這是一個簡單的故事：更多的抽象是好的；更少的抽象是差的。

Of course, it is not that simple. The landscape here is complex, with multiple offerings. In [“Taming the Compute Environment” on page 518](#_bookmark2134), we discussed the progression from dealing with pets running on bare-metal machines (either owned by your organization or rented from a colocation center) to managing containers as cattle. In between, as an alternative path, are VM-based offerings in which VMs can progress from being a more flexible substitute for bare metal (in Infrastructure as a Service offerings like Google Compute Engine [GCE] or Amazon EC2) to heavier substitutes for containers (with autoscaling, rightsizing, and other management tools).

當然，事情沒有那麼簡單。這裡的情況很複雜，有多種產品。在第518頁的 "馴服計算環境"中，我們討論了從處理在裸機上執行的寵物（無論是你的組織擁有的還是從主機託管中心租用的）到管理容器的進展情況。在這兩者之間，作為一個替代路徑，是基於虛擬機器的產品，其中虛擬機器可以從更靈活地替代裸機（在基礎設施即服務產品中，如谷歌計算引擎[GCE]或亞馬遜EC2）發展到更重地替代容器（具有自動伸縮、許可權調整和其他管理工具）。

In Google’s experience, the choice of managing cattle (and not pets) is the solution to managing at scale. To reiterate, if each of your teams will need just one pet machine in each of your datacenters, your management costs will rise superlinearly with your organization’s growth (because both the number of teams *and* the number of datacenters a team occupies are likely to grow). And after the choice to manage cattle is made, containers are a natural choice for management; they are lighter weight (implying smaller resource overheads and startup times) and configurable enough that should you need to provide specialized hardware access to a specific type of workload, you can (if you so choose) allow punching a hole through easily.

根據谷歌的經驗，選擇管理牛（而不是寵物）是規模管理的解決方案。重申一下，如果你的每個團隊在每個資料中心只需要一臺寵物機，那麼你的管理成本將隨著你的組織的增長而呈超線性上升（因為團隊的數量*和*一個團隊所佔用的資料中心的數量都可能增長）。而在選擇了管理牛之後，容器是管理的自然選擇；它們的重量更輕（意味著更小的資源開銷和啟動時間），而且可配置，如果你需要為特定型別的工作負載提供專門的硬體訪問，你可以（如果你選擇的話）允許輕鬆透傳透過。

The advantage of VMs as cattle lies primarily in the ability to bring our own operating system, which matters if your workloads require a diverse set of operating systems to run. Multiple organizations will also have preexisting experience in managing VMs, and preexisting configurations and workloads based on VMs, and so might choose to use VMs instead of containers to ease migration costs.

虛擬機器作為牛的優勢主要在於能夠帶來我們自己的作業系統，如果你的工作環境需要一組不同的作業系統來執行，這一點很重要。多個組織在管理虛擬機器、基於虛擬機器的現有配置和工作負載方面也有經驗，因此可能會選擇使用虛擬機器而不是容器來降低遷移成本。


#### What is serverless? 什麼是無伺服器？

An even higher level of abstraction is *serverless* offerings.[^23] Assume that an organization is serving web content and is using (or willing to adopt) a common server framework for handling the HTTP requests and serving responses. The key defining trait of a framework is the inversion of control—so, the user will only be responsible for writing an “Action” or “Handler” of some sort—a function in the chosen language that takes the request parameters and returns the response.

更高層次的抽象是無伺服器產品。假設一個組織正在為網路內容提供服務，並且正在使用（或願意採用）一個通用的伺服器框架來處理HTTP請求和提供響應。框架的關鍵定義特徵是控制權的倒置--因此，使用者只負責編寫某種 "行動 "或 "處理程式"--所選語言中的函式，接收請求引數並返回響應。

In the Borg world, the way you run this code is that you stand up a replicated container, each replica containing a server consisting of framework code and your functions. If traffic increases, you will handle this by scaling up (adding replicas or expanding into new datacenters). If traffic decreases, you will scale down. Note that a minimal presence (Google usually assumes at least three replicas in each datacenter a server is running in) is required.

在Borg的世界裡，你執行這段程式碼的方式是，你建立一個副本的容器，每個副本包含一個由框架程式碼和你的功能組成的伺服器。如果流量增加，你將透過擴大規模來處理（增加副本或擴充套件到新的資料中心）。如果流量減少，你將縮小規模。請注意，需要一個最小的存在（谷歌通常假設伺服器執行的每個資料中心至少有三個副本）。

However, if multiple different teams are using the same framework, a different approach is possible: instead of just making the machines multitenant, we can also make the framework servers themselves multitenant. In this approach, we end up running a larger number of framework servers, dynamically load/unload the action code on different servers as needed, and dynamically direct requests to those servers that have the relevant action code loaded. Individual teams no longer run servers, hence “serverless.”

但是，如果多個不同的團隊使用同一個框架，就可以採用不同的方法：不只是讓機器多租，我們還可以讓框架伺服器本身共享。在這種方法中，我們最終會執行更多的框架伺服器，根據需要在不同的伺服器上動態載入/解除安裝動作程式碼，並將請求動態地引導到那些載入了相關動作程式碼的伺服器。各個團隊不再執行伺服器，因此 "無伺服器"。

Most discussions of serverless frameworks compare them to the “VMs as pets” model. In this context, the serverless concept is a true revolution, as it brings in all of the benefits of cattle management—autoscaling, lower overhead, lack of explicit provisioning of servers. However, as described earlier, the move to a shared, multitenant,cattle-based model should already be a goal for an organization planning to scale; and so the natural comparison point for serverless architectures should be “persistent containers” architecture like Borg, Kubernetes, or Mesosphere.

大多數關於無伺服器框架的討論都將其與 "虛擬機器作為寵物 "的模式相比較。在這種情況下，無伺服器概念是一場真正的革命，因為它帶來了牛群管理的所有好處--自動擴充套件、較低的開銷、缺乏明確的伺服器配置。然而，正如前文所述，對於計劃擴充套件的組織來說，轉向共享、多租戶、基於牛的模式應該已經是一個目標；因此，無伺服器架構的自然比較點應該是 "永續性容器 "架構，如Borg、Kubernetes或Mesosphere。


> [^23]:	FaaS (Function as a Service) and PaaS (Platform as a Service) are related terms to serverless. There are differences between the three terms, but there are more similarities, and the boundaries are somewhat blurred./
> 23 FaaS（功能即服務）和PaaS（平臺即服務）是與無伺服器相關的術語。這三個術語之間有區別，但更多的是相似之處，而且邊界有些模糊不清。


#### Pros and cons 利與弊

First note that a serverless architecture requires your code to be *truly stateless*; it’s unlikely we will be able to run your users’ VMs or implement Spanner inside the serverless architecture. All the ways of managing local state (except not using it) that we talked about earlier do not apply. In the containerized world, you might spend a few seconds or minutes at startup setting up connections to other services, populating caches from cold storage, and so on, and you expect that in the typical case you will be given a grace period before termination. In a serverless model, there is no local state that is really persisted across requests; everything that you want to use, you should set up in request-scope.

首先要注意的是，無伺服器架構要求你的程式碼必須是*真正的無狀態*；我們不太可能在無伺服器架構內執行你使用者的虛擬機器或實現Spanner。我們之前談到的所有管理本地狀態的方法（除了不使用它）都不適用。在容器化的世界裡，你可能會在啟動時花幾秒鐘或幾分鐘的時間來設定與其他服務的連線，從冷資料中填充快取，等等，你期望在典型情況下，在終止前會有一個寬限期。在無伺服器模型中，不存在真正跨請求持久化的本地狀態；所有你想使用的東西，你都應該在請求範圍內設定。

In practice, most organizations have needs that cannot be served by truly stateless workloads. This can either lead to depending on specific solutions (either home grown or third party) for specific problems (like a managed database solution, which is a frequent companion to a public cloud serverless offering) or to having two solutions: a container-based one and a serverless one. It’s worth mentioning that many or most serverless frameworks are built on top of other compute layers: AppEngine runs on Borg, Knative runs on Kubernetes, Lambda runs on Amazon EC2.

在實踐中，大多陣列織的需求都無法由真正的無狀態工作負載來滿足。這可能會導致依賴特定的解決方案（無論是本地的還是第三方的）來解決特定的問題（比如管理資料庫的解決方案，這是公有云無伺服器產品的常見配套），或者擁有兩個解決方案：一個基於容器的解決方案和一個無伺服器的解決方案。值得一提的是，許多或大多數無伺服器框架是建立在其他計算層之上的。AppEngine執行在Borg上，Knative執行在Kubernetes上，Lambda執行在Amazon EC2上。

The managed serverless model is attractive for *adaptable scaling* of the resource cost, especially at the low-traffic end. In, say, Kubernetes, your replicated container cannot scale down to zero containers (because the assumption is that spinning up both a container and a node is too slow to be done at request serving time). This means that there is a minimum cost of just having an application available in the persistent cluster model. On the other hand, a serverless application can easily scale down to zero; and so the cost of just owning it scales with the traffic.

管理無伺服器模式對於資源成本的*適應性擴充套件*很有吸引力，特別是在低流量的一端。在Kubernetes中，你的容器不能縮容到零容器（因為假設在請求服務時間內，同時啟動容器和節點的速度太慢）。這意味著，在持久化叢集模型中，僅僅擁有一個應用程式是有最低成本的。另一方面，無伺服器應用程式可以很容易地縮容到零；因此，僅僅擁有它的成本隨著流量的增加而增加。

At the very high-traffic end, you will necessarily be limited by the underlying infrastructure, regardless of the compute solution. If your application needs to use 100,000 cores to serve its traffic, there needs to be 100,000 physical cores available in whatever physical equipment is backing the infrastructure you are using. At the somewhat lower end, where your application does have enough traffic to keep multiple servers busy but not enough to present problems to the infrastructure provider, both the persistent container solution and the serverless solution can scale to handle it, although the scaling of the serverless solution will be more reactive and more granular than that of the persistent container one.

在非常高的流量端，無論採用何種計算解決方案，您都必須受到底層基礎設施的限制。如果你的應用程式需要使用100,000個核心來服務於它的流量，那麼在你所使用的基礎設施的任何物理裝置中需要有100,000個物理核心可用。在較低端的情況下，如果你的應用有足夠的流量讓多個伺服器忙碌，但又不足以給基礎設施提供商帶來問題，那麼持久化容器解決方案和無伺服器解決方案都可以擴充套件來處理，儘管無伺服器解決方案的擴充套件將比持久化容器解決方案更具有高響應和細粒度。

Finally, adopting a serverless solution implies a certain loss of control over your environment. On some level, this is a good thing: having control means having to exercise it, and that means management overhead. But, of course, this also means that if you need some extra functionality that’s not available in the framework you use, it will become a problem for you.

最後，採用無伺服器解決方案意味著在一定程度上失去了對環境的控制。在某種程度上，這是一件好事：擁有控制權意味著必須行使它，而這意味著管理開銷。但當然，這也意味著，如果你需要一些你所使用的框架中沒有的額外功能，這將成為你的一個問題。

To take one specific instance of that, the Google Code Jam team (running a programming contest for thousands of participants, with a frontend running on Google AppEngine) had a custom-made script to hit the contest webpage with an artificial traffic spike several minutes before the contest start, in order to warm up enough instances of the app to serve the actual traffic that happened when the contest started. This worked, but it’s the sort of hand-tweaking (and also hacking) that one would hope to get away from by choosing a serverless solution.

舉個具體的例子，谷歌Code Jam團隊（為數千名參賽者舉辦的程式設計比賽，其前端執行在谷歌AppEngine上）有一個訂製的指令碼，在比賽開始前幾分鐘給比賽網頁帶來了人為的流量高峰，以便為應用程式的足夠實例預熱，為比賽開始時的實際流量提供服務。這很有效，但這是人們希望透過選擇無伺服器解決方案來擺脫的那種手工調整（也是黑客科技）。

#### The trade-off 權衡

Google’s choice in this trade-off was not to invest heavily into serverless solutions. Google’s persistent containers solution, Borg, is advanced enough to offer most of the serverless benefits (like autoscaling, various frameworks for different types of applications, deployment tools, unified logging and monitoring tools, and more). The one thing missing is the more aggressive scaling (in particular, the ability to scale down to zero), but the vast majority of Google’s resource footprint comes from high-traffic services, and so it’s comparably cheap to overprovision the small services. At the same time, Google runs multiple applications that would not work in the “truly stateless” world, from GCE, through home-grown database systems like [BigQuery ](https://cloud.google.com/bigquery)or Spanner, to servers that take a long time to populate the cache, like the aforementioned long- tail search serving jobs. Thus, the benefits of having one common unified architecture for all of these things outweigh the potential gains for having a separate serverless stack for a part of a part of the workloads.

谷歌在這種權衡的選擇是不對無伺服器解決方案進行大量投資。谷歌的持久化容器解決方案Borg足夠先進，可以提供大部分無伺服器的好處（比如自動伸縮、針對不同型別應用的各種框架、部署工具、統一的日誌和監控工具等等）。缺少的是更積極的擴充套件（特別是將規模縮小到零的能力），但谷歌的絕大部分資源足跡來自高流量服務，因此過度供應小服務的成本相對較低。同時，谷歌執行的多個應用程式在“真正無狀態”的世界中不起作用，從GCE，到自制的資料庫系統，如[BigQuery](https://cloud.google.com/bigquery)或Spanner，再到需要長時間填充快取的伺服器，如上述的長尾搜尋服務工作。因此，對所有這些事情采用一個共同的統一架構的好處超過了對一部分工作負載採用單獨的無伺服器方向的潛在收益。

However, Google’s choice is not necessarily the correct choice for every organization: other organizations have successfully built out on mixed container/serverless architectures, or on purely serverless architectures utilizing third-party solutions for storage.

然而，谷歌的選擇並不一定是每個組織的正確選擇：其他組織已經成功地建立了混合容器/無伺服器架構，或在純粹的無伺服器架構上利用第三方解決方案進行儲存。

The main pull of serverless, however, comes not in the case of a large organization making the choice, but in the case of a smaller organization or team; in that case, the comparison is inherently unfair. The serverless model, though being more restrictive, allows the infrastructure vendor to pick up a much larger share of the overall management overhead and thus *decrease the management overhead* for the users. Running the code of one team on a shared serverless architecture, like AWS Lambda or Google’s Cloud Run, is significantly simpler (and cheaper) than setting up a cluster to run the code on a managed container service like GKE or AKS if the cluster is not being shared among many teams. If your team wants to reap the benefits of a managed compute offering but your larger organization is unwilling or unable to move to a persistent containers-based solution, a serverless offering by one of the public cloud providers is likely to be attractive to you because the cost (in resources and management) of a shared cluster amortizes well only if the cluster is truly shared (between multiple teams in the organization).

然而，無伺服器的主要吸引力並不是來自於一個大型組織的選擇，而是來自於一個較小的組織或團隊；在這種情況下，這種比較本身就是不公平的。無伺服器模式雖然限制更大，但允許基礎設施供應商承擔更大的總體管理開銷，從而減少使用者的管理開銷。在共享的無伺服器體系結構（如AWS Lambda或Google的Cloud Run）上執行一個團隊的程式碼，要比在多個團隊之間不共享叢集的情況下，在GKE或AKS等託管容器服務上設定叢集來執行程式碼要簡單得多（而且更便宜）。如果你的團隊希望從託管計算產品中獲益，但你的公司不願意或無法轉向基於持久容器的解決方案，那麼由一家公共雲提供商提供的無伺服器產品可能會對你有吸引力，因為成本（資源和成本）很高只有當叢集真正共享（在組織中的多個團隊之間）時，共享叢集的管理才能很好地攤銷。

Note, however, that as your organization grows and adoption of managed technologies spreads, you are likely to outgrow the constraints of a purely serverless solution. This makes solutions where a break-out path exists (like from KNative to Kubernetes) attractive given that they provide a natural path to a unified compute architecture like Google’s, should your organization decide to go down that path.

但是，請注意，隨著組織的發展和託管技術的普及，你很可能會超越純無伺服器解決方案的限制。這使得存在突破路徑的解決方案（如從KNative到Kubernetes）具有吸引力，因為如果您的組織決定走這條路，它們提供了一條通向像Google這樣的統一計算體系結構的自然路徑。

### Public Versus Private 公有與私有

Back when Google was starting, the CaaS offerings were primarily homegrown; if you wanted one, you built it. Your only choice in the public-versus-private space was between owning the machines and renting them, but all the management of your fleet was up to you.

當谷歌剛剛起步時，CaaS產品主要是本土產品；如果你想要一個，你就建造它。在公共空間和私有空間中，你唯一的選擇是擁有機器和租用機器，但你的叢集的所有管理都取決於你。

In the age of public cloud, there are cheaper options, but there are also more choices, and an organization will have to make them.

在公有云時代，有更便宜的選擇，但也有更多的選擇，組織必須做出選擇。

An organization using a public cloud is effectively outsourcing (a part of) the management overhead to a public cloud provider. For many organizations, this is an attractive proposition—they can focus on providing value in their specific area of expertise and do not need to grow significant infrastructure expertise. Although the cloud providers (of course) charge more than the bare cost of the metal to recoup the management expenses, they have the expertise already built up, and they are sharing it across multiple customers.

使用公共雲的機構實際上是將管理費用（部分）外包給公共雲供應商。對於許多組織來說，這是一個有吸引力的提議--他們可以專注於在其特定的專業領域提供價值，而不需要增加重要的基礎架構專業知識。雖然雲供應商（當然）收取的費用超過了裸機的最低成本，以收回管理費用，但他們已經建立了專業知識，並在多個客戶之間共享。

Additionally, a public cloud is a way to scale the infrastructure more easily. As the level of abstraction grows—from colocations, through buying VM time, up to managed containers and serverless offerings—the ease of scaling up increases—from having to sign a rental agreement for colocation space, through the need to run a CLI to get a few more VMs, up to autoscaling tools for which your resource footprint changes automatically with the traffic you receive. Especially for young organizations or products, predicting resource requirements is challenging, and so the advantages of not having to provision resources up front are significant.

此外，公共雲是一種更容易擴充套件基礎設施的方式。隨著抽象水平的提高--從主機託管，到購買虛擬機器時間，再到管理容器和無伺服器產品--擴充套件的難度也在增加--從必須簽署主機託管空間的租賃協議，到需要執行CLI來獲得更多的虛擬機器，再到自動擴充套件工具，你的資源足跡隨著你收到的流量自動變化。特別是對於年輕的組織或產品，預測資源需求是具有挑戰性的，因此，不必預先配置資源的優勢是非常顯著的。

One significant concern when choosing a cloud provider is the fear of lock-in—the provider might suddenly increase their prices or maybe just fail, leaving an organization in a very difficult position. One of the first serverless offering providers, Zimki, a Platform as a Service environment for running JavaScript, shut down in 2007 with three months’ notice.

在選擇雲端計算供應商時，一個重要的顧慮是擔心被鎖定--供應商可能會突然漲價，或者直接倒閉，讓企業陷入非常困難的境地。最早的無伺服器提供商之一Zimki，一個執行JavaScript的平臺即服務環境，在2007年關閉，只提前三個月通知。

A partial mitigation for this is to use public cloud solutions that run using an open source architecture (like Kubernetes). This is intended to make sure that a migration path exists, even if the particular infrastructure provider becomes unacceptable for some reason. Although this mitigates a significant part of the risk, it is not a perfect strategy. Because of Hyrum’s Law, it’s difficult to guarantee no parts that are specific to a given provider will be used.

對此的部分緩解措施是使用使用開源架構（如Kubernetes）執行的公共雲解決方案。這是為了確保存在一個遷移路徑，即使特定的基礎設施供應商由於某種原因變得不可接受。雖然這減輕了很大一部分風險，但這並不是一個完美的策略。由於海勒姆定律，很難保證不使用特定供應商的特定部分。

Two extensions of that strategy are possible. One is to use a lower-level public cloud solution (like Amazon EC2) and run a higher-level open source solution (like OpenWhisk or KNative) on top of it. This tries to ensure that if you want to migrate out, you can take whatever tweaks you did to the higher-level solution, tooling you built on top of it, and implicit dependencies you have along with you. The other is to run multicloud; that is, to use managed services based on the same open source solutions from two or more different cloud providers (say, GKE and AKS for Kubernetes). This provides an even easier path for migration out of one of them, and also makes it more difficult to depend on specific implementation details available in one one of them.

這一戰略有兩種可能的擴充套件。一種是使用較低級別的公有云解決方案（如亞馬遜EC2），並在其上執行較高級別的開源解決方案（如OpenWhisk或KNative）。這試圖確保如果你想遷移出去，你可以帶著你對高階解決方案所做的任何調整，你在它上面建立的工具，以及你擁有的隱性依賴。另一種是執行多雲；也就是說，使用基於兩個或多個不同的雲供應商的相同開源解決方案的管理服務（例如，Kubernetes的GKE和AKS）。這為遷移出其中一個提供了更容易的路徑，同時也使你更難依賴其中一個的具體實施細節。

One more related strategy—less for managing lock-in, and more for managing migration—is to run in a hybrid cloud; that is, have a part of your overall workload on your private infrastructure, and part of it run on a public cloud provider. One of the ways this can be used is to use the public cloud as a way to deal with overflow. An organization can run most of its typical workload on a private cloud, but in case of resource shortage, scale some of the workloads out to a public cloud. Again, to make this work effectively, the same open source compute infrastructure solution needs to be used in both spaces.

還有一個相關的策略--不是為了管理鎖定，而是為了管理遷移--是在混合雲中執行；也就是說，在你的私有基礎設施上有一部分整體工作負載，而在公共雲供應商上執行一部分。其中一種方法是使用公共雲來處理多出的資源需求。一個組織可以在私有云上執行其大部分典型的工作負載，但在資源短缺的情況下，將一些工作負載擴充套件到公共雲上。同樣，為了使其有效運作，需要在兩個空間使用相同的開源計算基礎設施解決方案。

Both multicloud and hybrid cloud strategies require the multiple environments to be connected well, through direct network connectivity between machines in different environments and common APIs that are available in both.

多雲和混合雲戰略都需要將多個環境很好地連線起來，透過不同環境中的機器之間的直接網路連線和兩個環境中都有的通用API。

## Conclusion 總結

Over the course of building, refining, and running its compute infrastructure, Google learned the value of a well-designed, common compute infrastructure. Having a single infrastructure for the entire organization (e.g., one or a small number of shared Kubernetes clusters per region) provides significant efficiency gains in management and resource costs and allows the development of shared tooling on top of that infrastructure. In the building of such an architecture, containers are a key tool to allow sharing a physical (or virtual) machine between different tasks (leading to resource efficiency) as well as to provide an abstraction layer between the application and the operating system that provides resilience over time.

在建構、完善和執行計算基礎設施的過程中，谷歌認識到了設計良好的通用計算基礎設施的價值。為整個組織提供單一的基礎設施（例如，每個區域一個或少數共享Kubernetes叢集）可以顯著提高管理效率和資源成本，並允許在基礎設施之上開發共享工具。在建構這樣一個體繫結構時，容器是一個關鍵工具，它允許在不同的任務之間共享物理（或虛擬）機器（從而提高資源效率），並在應用程式和作業系統之間提供一個抽象層，隨著時間的推移提供彈性。

Utilizing a container-based architecture well requires designing applications to use the “cattle” model: engineering your application to consist of nodes that can be easily and automatically replaced allows scaling to thousands of instances. Writing software to be compatible with that model requires different thought patterns; for example, treating all local storage (including disk) as ephemeral and avoiding hardcoding hostnames.

充分利用基於容器的體系結構需要設計使用“牛”模型的應用程式：將應用程式設計為由可以輕鬆自動替換的節點組成，從而可以擴充套件到數千個實例。編寫與該模型相容的軟體需要不同的思維模式；例如，將所有本地儲存（包括磁碟）視為短暫的，並避免硬編碼主機名。

That said, although Google has, overall, been both satisfied and successful with its choice of architecture, other organizations will choose from a wide range of compute services—from the “pets” model of hand-managed VMs or machines, through “cattle” replicated containers, to the abstract “serverless” model, all available in managed and open source flavors; your choice is a complex trade-off of many factors.

這就是說，儘管谷歌總體上對其架構的選擇感到滿意並取得了成功，但其他組織將從一系列計算服務中進行選擇，從手工管理的虛擬機器或機器的“寵物”模型，透過“牛”複製容器，到抽象的“無伺服器”模型，所有版本都有託管和開源版本；你的選擇是許多因素的複雜權衡。

## TL;DRs  內容提要

- Scale requires a common infrastructure for running workloads in production.
- A compute solution can provide a standardized, stable abstraction and environment for software.
- Software needs to be adapted to a distributed, managed compute environment.
- The compute solution for an organization should be chosen thoughtfully to provide appropriate levels of abstraction.

- 規模化需要一個通用的基礎設施來執行生產中的工作負載。
- 一個計算解決方案可以為軟體提供一個標準化的、穩定的抽象和環境。
- 軟體需要適應一個分散式的、可管理的計算環境。
- 組織的計算解決方案應經過深思熟慮的選擇，以提供適當的抽象級別。

