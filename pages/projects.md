---
layout: page
title: Projects
permalink: /projects/
---

Projects/Products I designed and developed from the scratch or contributed 
partially over the past years in full-time, part-time, freelance/remote or 
consultancy are below - 

## Chronology

### 2025

- [OICM](https://openinnovation.ai/open-innovation-cluster-manager/) — *Open Innovation AI, Abu Dhabi*
    - OICM is the flagship AI/ML platform of Open Innovation AI, designed to orchestrate and manage large-scale machine learning workloads across distributed GPU clusters. 
    - I lead the **Platform Engineering & MLOps** initiatives — designing and maintaining distributed microservices for AI/ML orchestration, GPU scheduling, and resource quota management.
    - Key contributions include:
        - Building a **multi-cluster GPU monitoring system** with Prometheus, DCGM, and Loki.
        - Developing **tenant-aware storage and resource quota management** for NFS/OBS/MinIO/FlashBlade.
        - Implementing **custom exporters and controllers** for GPU metrics, workload status, and event persistence.
        - Designing CI/CD pipelines with Helm, Terraform, and GitLab CI for cluster-wide deployments.
    - Tech Stack: `Kubernetes`, `Helm`, `Terraform`, `Python`, `Go`, `Prometheus`, `Loki`, `Grafana`, `Flask`, `FastAPI`, `MongoDB`, `PostgreSQL`, `Ray`, `Volcano`, `NVIDIA DCGM`, `GitLab CI`, `vLLM`, `TGI`, `SGLang`.

- [GitGossip](https://github.com/osmangoninahid/gitgossip)
    - GitGossip is an open-source CLI tool that generates **AI-powered human-readable summaries** of Git commits, repository activities, and merge requests.
    - I designed the project architecture with a modular service layer, focusing on testability, SOLID design, and LLM integration for semantic summarization.
    - Core components include:
        - `RepoDiscoveryService` for detecting repositories and commits.
        - `CommitParser` for structured commit analysis.
        - `SummarizerService` and `ILLMAnalyzer` for natural language summaries.
        - CLI built using `Typer`, integrated with `Rich` for a polished terminal experience.
    - Tech Stack: `Python`, `Typer`, `Pydantic v2`, `GitPython`, `Rich`, `pytest`, `Ruff`, `Mypy`, `Black`, `LLM APIs`, `OpenAI`

### 2023 to 2024

- [MizMiz](https://mizmiz.com/)
    - MizMiz is a TikTok-like short video content platform focused on user-generated video sharing and engagement.
    - I worked on the **video transcription pipeline**, enabling automatic caption generation and metadata extraction for uploaded videos to enhance accessibility and searchability.
    - Tech Stack: `Python`, `NestJS`, `FFmpeg`, `AWS-CloudFront`, `AWS S3`, `AWS-Neptune`, `Redis`, `GraphQL`, `Docker`, `Kubernetes`

- [ComeraPay](https://comerapay.com/)
    - ComeraPay stands as Comera's mobile wallet application, providing services including fund transfers,
      cash deposits, electronic transactions, bill payments, and remittance sending.
    - I oversee the Backend Architecture, Solution Design and Guiding & reviewing backend code.
    - Tech Stack: `Microservice`, `NodeJs/ExpressJs`, `MySQL`, `Third-Party Payment Integration`, `Docker`,
       `AWS`, `Jenkins`, `NFC`  

- [Comera](https://mycomera.com/)
    - A local market-oriented application in the UAE competing with platforms like WhatsApp and Telegram,
      providing services such as voice calls, video calls, conference calls, instant messaging, and more.
    - I design solutions and oversee the XMPP-Messaging segment, also leading the Sustenance Team.
    - Tech Stack: `XMPP`, `MongooseIM`, `Erlang`, `Node.js`, `MySQL`, `Cassandra`, `AWS`, `Jenkins`, `WebRTC`

### 2021 to 2022

- [rDash360](https://rdash360.com/)
    - rDash360 serves as a comprehensive solution for overseeing inventory, orders, content, and logistics
      across various online marketplaces like Shopee, Tokopedia, Lazada, Blibli, Bukalapak, and more. 
      It empowers merchants and large brands to adeptly handle their omnichannel online sales, providing
      seamless management capabilities akin to professional expertise.
    - Led the team of 5 resources, Designed and Developed Backend Microservices also contributed on Deployment.
    - Tech Stack: `Microservice`, `NodeJs/NestJs`, `PostgreSQL`, `MongoDB`, `RabbitMQ`, `Redis`, `AWS`
      `Docker`, `Kubernetes`, `Jenkins`

- [RAENA](https://play.google.com/store/apps/details?id=com.raenaapp)
    - RAENA is the largest beauty products reselling & dropship app in Indonesia, trusted by over 100,000
      beauty product sellers. RAENA supports Beautypreneur in starting and growing beauty business without
      any upfront capital and earn extra income up to Rp 10 million/month.
    - Worked as Lead Backend Engineer, Contributed on core Backend Service Design & Development.
    - Tech stack: `NodeJs/ExpressJs`, `NestJs/Typescript`, `PostgreSQL`, `Redis`, `RabbitMQ`, `AWS`
      `Docker`, `Kubernetes`, `Jenkins`.

### 2020 to 2021


- [eJob](https://ejob.com.bd)
    - eJob functions as a job portal catering to both job seekers and employers, offering a platform
      to post and discover suitable job opportunities. Additionally, it provides services such as profile
      and resume building tools, as well as employer reviews, enhancing the overall job-seeking experience.
    - Led the over-all product and team, Designed the solution from scratch.
    - Tech Stack: `Python/Django`, `PostgreSQL`, `ElasticSearch`, `Redis`, `AWS`, `Docker`, `Kubernetes`,
     `Jenkins`, `ReactJs`, `NextJs`.
    
- [eLogistic](https://elogistics.com.bd)
    - eLogistic serves as a comprehensive fleet and logistics aggregator, seamlessly integrating Evaly's internal
      operations with third-party logistics (3PL) and partners. It facilitates various delivery services
      such as on-demand, same-day, express, and regular trucking through a unified platform.
    - Led the over-all product and team, Designed the solution from scratch.
    - Tech Stack: `Microservice`, `Event-Driven Architecture`, `Kafka`, `Redis`, `MongoDB`, `PostgreSQL`,
       `AWS`, `Docker`, `Kubernetes`, `NodeJs/NestJs`, `RedShift`, `Jenkins`

- [eHealth](https://play.google.com/store/apps/details?id=bd.com.evaly.ehealth)
    - eHealth is the first digital one-stop health solution in Bangladesh. A sister concern of Evaly.com Limited,
      this application brings the 21st century care to your fingertips. This app will assist you in a comprehensive
      and convenient way to operate your health-related needs.
    - Led the Product also worked on Solution Design, Backend Development and Deployment.
    - Tech Stack: `Microservice`, `AWS`, `Docker`, `Kubernetes`, `Golang`, `NodeJS`, `MongoDB`, `PostgreSQL`,
      `Redis`, `RabbitMQ`, `Jenkins`

- [eKhata](https://play.google.com/store/apps/details?id=bd.com.evaly.ekhata)
    - eKhata offers a digital ledger cash book service, providing SME businesses with a free, safe,
      and secure platform to manage customer accounts. Also shop owners to record credit and debit 
      transactions for their loyal customers.
    - Co-Founded and Led the Product also worked on Solution Design, Backend Development and Deployment.
    - Tech Stack: `Microservice`, `AWS`, `Docker`, `Kuberenetes`, `NodeJS`, `MongoDB`, `PostgreSQL`, `Redis`
      `AzureDevOps`

### 2019 to 2020

- [Evaly](https://play.google.com/store/apps/details?id=bd.com.evaly.evalyshop)
    - Evaly holds the title as Bangladesh's largest and most favored e-commerce marketplace,
      trusted by over 7 million users. It provides a wide array of goods and products, spanning
      from groceries to high-end luxury vehicles.
    - Joined as founding Engineer. Hired, built and mentored team of 100+ resources, Designed the overall
      system from scratch, developed & scaled 30+ microservices and successfully handled 1+ million request per minute. 
    - Beside playing the role of CTO, actively worked as Backend Engineer and Led the Backend Team. 
    - Tech Stack: `Event-Driven Microservice`, `Python/Django`, `NodeJs/NestJs`, `Golang`, `MySQL`, `PostgreSQL`, `MongoDB`,
     `RedShift`, `ElasticSearch`, `Kafka`, `Redis`, `RabbitMQ`, `AzureDevOps`, `Jenkins`, `AWS`, `Docker`
      `Kubernetes`.
- [eFood](https://efood.live)
    - eFood stands as an on-demand food delivery platform that consistently handles the highest or 
      second-highest volume of orders nationwide among its competitors. With a user base of 4 million,
      it efficiently manages over 100 orders per minute during peak hours.
    - Contributed on Backend Architecture Design and Deployment.
    - Tech Stack: `Microservice`, `Python`, `NodeJs`, `Django`, `Asyncio`, `PostgreSQL`, `MongoDB`, 
      `ElasticSearch`, `Redis`, `RabbitMQ`, `Kafka`, `AzureDevOps`, `Docker`, `Kubernetes`, `AWS`

- [eConnect](https://play.google.com/store/apps/details?id=bd.com.evaly.econnect)
    - eConnect is an instant messaging application, built on XMPP, exclusively for users on Evaly's platform.
      It enables 1-1 and group chats, doubling as a evaly customer support tool by efficiently assigning and 
      managing customer service agents. 
    - Designed and Developed the backend (ejabberd,REST microservie) from scratch also contributed on deployment.
    - Tech Stack: `XMPP`, `Ejabberd`, `Microservice`, `NodeJs/ExpresJs`, `Mnesia`, `MySQL`, `AWS`

- [eBazar](https://ebazar.com.bd)
    - eBazar is a C2C resale platform designed for buying and selling both new and pre-owned
      items across various categories. Another concern of Evaly.
    - Co-Founded and Led the Product also worked on Solution Design, Backend Development and Deployment.
    - Tech Stack: `Python/Django`, `PostgreSQL`, `ElasticSearch`,`Redis`, `AzureDevOps`, `AWS`, `Docker`,
      `Kubernetes`, `ReactJs`, `NestJs`.

    

### 2018 to 2019

- [ekkHero](https://play.google.com/store/apps/details?id=com.ekkbaz.hero)
    - To Track business employee, sales re-presentative's KPI and take order from buyer for ekkBaz Business.
    - Led the team of 5 resources, Designed and Developed Backend Architecture.
    - Tech Stack: `Python`, `Django`, `Sanic`, `Redis`, `RabbitMQ` `MongoDB`, `PostgreSQL` `PostGIS`,
     `Azure`, `Microservice`, `GoogleMaps`

- [Tradester](https://tradester.com)
    - Tradester offers the platform to create a more inclusive and empowered working class community,
      where individuals have access to resources, opportunities, and support networks that enable personal
     and professional growth.
    - Worked as Full Stack Developer
    - Tech Stack : `NodeJS/ExpressJS`, `MongoDB`, `REST`, `SocketIO`, `ReactJS`, `AWS`, `Docker`


- [mCare](https://mcare.com)
    - mCare offers to create and coordinate a village of care around your loved one in one central place.
      Invite whomever you like to join your group and quickly manage everything from scheduling rides and meals
     to appointments and family gatherings. Monitor progress of medication, manage caregiver etc.
    - Led team and worked on Backend Development mostly.
    - Tech Stack: `NodeJS`, `PostgreSQL`, `Redis`, `REST`, `SocketIO`, `Docker`, `Nginx`, `AWS`

### 2017 to 2018

- [ekkBaz](https://ekkbaz.com)
    - EkkBaz is a B2B trade platform, designed specifically for small & medium businesses.
      It brings traders, wholesalers, retailers and manufacturers in one single platform for ease of trading.
    - Led and managed 20 resources, Solution Designed and Backend Development.
    - Tech Stack: `NodeJS/ExpressJS`, `Python/Sanic`, `Redis`, `AzureCosmos`, `MongoDB`, `Table Storage`,
     `Microservice`, `REST`, `Azure Cloud`, `Azure Function`, `Docker`, `Nginx`

- [ekkChat](https://play.google.com/store/apps/details?id=com.ekkbaz.business)
    - ekkChat enabled (XMPP) instant messenging for business to business, business to customer, small business to 
      sales representatives through XMPP.
    - Led and managed team of 4 resources, Worked on Backend Design & Development in REST and XMPP/Ejabberd.
    - Tech Stack: `NodeJS/ExpressJS`, `MySQL`, `MongoDB`, `Ejabberd`, `XMPP`, `Mnesia`, `Azure`, `Nginx`, `Docker`

- [Defense Tracker](https://en.wikipedia.org/wiki/Non-disclosure_agreement) `Can't disclose the link or exact name due to NDA`
    - This application is being used by a country's National Security Intelligence team to track, monitor
    activities in training camp. Also provide visually summarize data for weekly or monthly. 
    - Worked in Backend Solution Design & Development
    - Tech Stack: `NodeJS`, `SocketIO`, `PostgreSQL`, `MongoDB`, `MQTT`, `Redis`, `IoT`

### 2016 to 2017

- [TeachFlex](https://teachflex.com)
    - TeachFlex is a live online tutoring tool that allows teachers to provide live tutoring to students
      through real-time technologies. For generations - students have been seeking in person tutoring to grasp concepts.
      And the delivery of tutoring has been using traditional tools like - abacus, pen, paper, white & blackboards.
    - Primarily worked in Backend Architecture Design & Development, later led the project and co-operated
      with frontend team and managed deployments
    - Tech Stack: `NodeJS/ExpressJS`, `MongoDB`, `PubNub`, `AWS`, `Socket.IO`, `ReactJS`, `Python-Fabric`

- [FinWallet](https://play.google.com/store/apps/details?id=pseudozero.smalam.com.finwallet)
    - FinWallet is SBAC Bank's digital wallet app enabling account holders to manage various banking
      activities seamlessly. Users can perform tasks such as fund transfers, bill payments, remittance,
      access account statements, and even withdraw cash directly through the app.
    - Led the cross-functional team and Worked in Backend Development.
    - Tech Stack: `Python/Django`, `MySQL`, `Python Fabric`, `AWS`, `supervisor`, `Nginx`

- [CollabHero](https://collabhero.com)
    - CollabHero is the easiest way for students to collaborate and give feedback on each other’s work
      and group contributions. CollabHero allows educators to create peer feedback tasks for students 
      to reflect and evaluate work of their peers.
    - Led and Worked primarily in Backend Development also contributed on Frontend as well.
    - Tech Stack: `Python`, `Django`, `PostgreSQL`, `Python Fabric`, `AWS`, `Gunicorn`, `Nginx`, `ReactJS`

- [Akly](http://akly.co/)
    - Akly is a platform for consumers who want healthy meals delivered to them in their home, office, or anywhere they are
      for any period of time. Akly provides 200 different healthy meals that users can mix and match to fit into their schedule - weekly, monthly, or as they like. Users can also purchase meal plans exclusively curated, such as “Paleo”, “Ketogenic”, “Low Carb” etc. Through this app, busy professionals and many others will get to enjoy a healthy lifestyle without any hassle and in a fast and convenient way.
    - Primarily worked in Backend and contributed on Frontend and Deployment.
    - Tech Stack: `NodeJS/ExpressJS`, `MongoDB`, `PM2`, `Nginx`, `AWS`, `Swift/iOS`, `ReactJS`, `Redux`, `Java/Android`,
                `PubNub`, `GCM`

- [LocalInsights](https://localinsights.io/)
    - LocalInsights initially began as a mortgage company but pivoted towards becoming a 
      successful data provider after struggling in the mortgage business. Recognizing the potential
      in aggregating public data, they transformed themselves and thrived by leveraging this data as a competitive edge.
    - Worked asb(Remote) Backend Engineer and Led the Backend team of 3 resources.
    - Tech stack: `NodeJS/ExpressJS`, `MongoDB`, `Python/Django`, `PostGIS`, `GoogleMaps`, `Docker`, `AWS`, `Lambda`

### 2015 to 2016

- [DriverAnywhere](https://play.google.com/store/apps/details?id=com.limo.driverphase2)
    - DriverAnywhere allows limousine and livery drivers to manage trips, billing,
      and availability as well as interact directly with dispatch. The app provides both 
      scheduled and on-demand trip functionality.
    - Worked as part-time freelance Backend Engineer through agency and contributed on deployment.
    - Tech Stack: `NodeJS`, `Python`, `MongoDB`, `PostGIS`, `RabbitMQ`, `Redis`, `GoogleMaps SDK`

- [DrugBD](https://drugbd.com)
    - DrugBD pioneers the realm of medicine e-commerce, facilitating online orders and 
      swift home deliveries of medications. It boasts a 24/7 emergency operation and delivery service,
      along with health and medicine reminders, effortless medicine refills/reorders, and the capability
      to monitor progress seamlessly.
    - Primarily worked in Backend Development and Deployment from scratch also Collaborated on Frontend.
    - Tech Stack: `NodeJS/ExpressJS`, `MongoDB`, `Redis`, `Nginx`, `PythonFab`, `DigitalOcean`, `Firebase` 

- [ASDetect](https://play.google.com/store/apps/details?id=au.edu.latrobe.asdetect)
    - ASDetect enables parents and caregivers to review possible early signs of autism in their children aged 
      younger than 2.5 years. With actual clinical videos of children with and without autism, each question focuses on a specific ‘social communication’ behaviour, for example, pointing, social smiling.
    - Freelance Project, Worked in Backend Team as Freelance Backend Engineer through third-party agency.
    - Tech Stack: `NodeJS/ExpressJS`, `MQTT`, `Redis`, `MongoDB`, `REST`, `PM2`.

### 2014 to 2015

- [VisionIPTV](https://play.google.com/store/apps/details?id=com.powergroupbd.bdiptv)
    - Vision IPTV offers local and international Live TV Streamming channels including sports, enternatinment
      and movie archives.
    - Worked as Full Stack to Develop and Design both backend and frontend applications from scratch
    - Tech Stack: `Python`, `Django`, `Java/Android`, `MySQL`, `DigitalOcean`, `Nginx`, `Gunicorn`, `Firebase`

- [Safaree](http://safaree.net)
    - Safaree offers a subscription-based entertainment mobile app tailored for Bangladeshi Expats
      residing in KSA and UAE. The services encompass a wide array of features such as local and
      international TV channels, FM channels, hotlines, and access to health emergency services.
    - Designed and Developed Backend Monolith Services and Subscription module from scratch.
    - Tech Stack: `Python`, `Django`, `MySQL`, `DigitalOcean`, `Nginx`, `Gunicorn`, `Firebase`, `Google Map SDK`


