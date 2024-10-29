---
title: Software Engineering at Google
date: 2024-09-29
lastmod: 2024-09-29
author:
  - Jimmy Lin
tags: 
draft: false
---
## Reference
- [Software Engineering at Google](https://qiangmzsx.github.io/Software-Engineering-at-Google/#/)
## What is Software Engineering ?
- 軟體的生命週期
	- how long will the code continue to be built, executed, and maintained? How long will this software provide value?
- Software engineering” differs from “programming” in dimensionality: programming is about producing code. Software engineering extends that to include the maintenance of that code for its useful life span
- Hyrum's Law 海勒姆定律
	- With a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody
## 如何融入團隊
- 你不是天才，就算你是天才，你也必須懂得如何團隊合作，切勿閉門造車
- Fail early, fail fast, fail often. 儘早測試，不斷迭代
## 如何領導團隊
- 組織架構
	- 大型且成熟的團隊通常會有：The Engineer Manager(has technical background) + The Tech Lead
	- 小型和新生的團隊通常會有：The Tech Lead Manager（等於 EM + TL），一人幹兩人份的活
- 如何成為領導
	- 在你職業生涯的某個階段，當你開始熟悉公司的各項業務，開始嘗試解決組員之間的衝突，開始為公司產品的開發負起更多責任，領導的機會自然而然的會降臨在你身上
- 領導該做些什麼事情
	- 一個好的領導會信任他的組員並充分授權，一個壞的領導會表現得像一個直升機父母一樣
	- 必須及時且有效的拯救一個低效的組員，因為如果不解決低效的問題，整個團隊都會被他拖累，且高效的員工會傾向於離開，而低效的員工既不會也沒有能力離開
	- 領導必須保持冷靜，因為你的一言一行都會被放大檢視，隨著你的領導階級越高，你言行的影響力會成倍數的放大
	- 領導的責任是當下組員遇到問題時，為他介紹適合解決這個問題的人。你不必知道每個問題的答案，你只需要知道誰知道如何解決這個問題
	- Software Engineering is some sense, is a pain allocation task, so as a manager, your job is to make sure the pain is allocated equally among your team.
	- 在 1-on-1 中關注每個組員的需求，每個人都會有需求，不論是短期或是長期（職涯規劃），這關乎到每個人的幸福感
	- 擔任團隊和外界（更上層管理層或是其他團隊）之間資訊的過濾器，將混亂的資訊，一直在變動的人事物與決定轉換成有秩序，有明確步驟與方向的任務給你的組員
	- 給組員自主性、掌控力和目標，他們把事情做好的內在動機就會變強
- 打造一個當你休假時，還能夠正常運作的團隊
- 領導和工程師的差別在於，領導負責決定方向，決定戰略，細節實現就交給手下的工程師
- 不要害怕被取代，而是要積極尋找你的接班人，當你找到之後，你就可以往下一份領導職位前進了，可以是別的組，也可以再往上晉升
- 成功會為團隊帶來更多的工作，作為領導，最重要的是要保護好自己的精力和注意力，因為你的工作不再是像工程師那樣有條有序地解 issue，而是到處救火，你的通訊軟體會被各種訊息塞滿
## Code Reivew
- Why Code Reivew
	- more robust
	- better code quality -> easier to maintain
	- 用一套既定的規則給予工程師肯定，避免工程師過於驕傲，或產生冒牌症候群，
- LGTM
	- correctness
	- readibility
	- 是否增加了未來的技術債？
	- 是否難以維護？
- Ownership
	- Project Owner -> distributed, no need for a centralized code base control
## Documentation
- 包括文件、註釋
- 文件
	- 文件的形式
		- XXWiki is not a good idea
			- 雖然文件是自由的，任何人都可以編輯，但是這也代表，文件沒有明確的 Owner，因此也沒有人去維護，文件品質很差
		- 將 Documentation 當作一個 code base，用 Git 和 project owner 去保證其品質
	- What makes a *good* documentation
		- TL;DR
		- Know your audience
		- keep it short by first writing a long version and then trim it down, 刪掉冗詞贅字以及重複的內容，要記住：less is more，理查．費曼 Richard Feynman 說過：如果你無法用簡單的語言解釋一件事，那就代表你並未真正理解它
		- WHO, WHAT, WHEN, WHERE, and WHY
		- 
	- Single Responsibility
		- one file for one single responsiblity
	- Tutorial
		- 新手教學，就是 example，而寫 tutorial 最好的時機點就是你剛加入團隊的時候，透過 try and error，然後記錄下最後讓事情 work 的方法和過程
- 註釋
	- 應該包含兩部分
		- 開頭說明：讓搜尋者可以輕鬆找到要找的文件（比如說用 grep），比如說在一個 header file 最一開始聲明這個 header file 提供什麼功能
		- 每個 function、class 的用法：讓使用者（也就是搜尋者找到了對的 header file 之後）可以輕鬆了解該如何使用
## Test

