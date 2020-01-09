---
title: 'Process Monitoring using IIoT'
date: Thu, 01 Jan 2020 12:59:09 +0000
draft: false
tags: [Consultancy, Project, IIoT, Software ]
---



### Problem Definition
 •   Machine with real-time process and production reporting has never been accurate and convenient. Accurate and timely process, production         reporting is key to the management decisions making process. Also process planning and scheduling can be improved. 

 •   In today’s manufacturing scenario huge losses/wastage occurs in the manufacturing shop floor. This wastage may be because of operator           negligence, improper maintenance, inefficient process, tooling problems and non-availability of components in the correct time. 

 •   Information must be received directly from shop floor machine when it is needed. Real-time monitoring will give the information about what      has been machined, what is under process and what has not been machined. 


###  Objectives
    •   Remotely monitor the real-time machine process and status using IoT. 
    •	Remotely monitor the reason for machine breakdown and time till machine is not working 
    •	Compute machine utilization and the Overall Equipment Effectiveness


### Hardware and Software 
The image shows the hardware mounted onto the machine

{{< figure src="/static/images/km1_hardware.png"  >}}


Design of the UI was done with discussion with the end user at KennaMetal. We made sure that the UI is simple, fast and easy to use. The below image show the Login page of the operator screen. The Operator before starting the machining process has to login into the IIOT. The IIOT device is programmed such a way that a lock signal is sent to the EcoGrind Machine. This stops the machine from starting. Every operator has his own unique credentials with password

{{< figure src="/static/images/km_login.png"  >}}


Once logged into the IIOT the below image Operator dashboard is displayed. We made sure that only necessary information is shown to the operator. The information is colour coded. The yellow box has a list of dropdown, the operator needs to select the part which he will machining. The parts are assigned to each operator by the supervisor The blue box displays downtimes which are incurred during the machining process. The data is automatically stored to database. If the operator crosses any downtime limit set by the operator He/she needs to enter the reason into the red box. The IIOT system locks the CNC machine and will not start unless a reason is entered. Today’s progress is single bar graph which show how many parts have been done.


{{< figure src="/static/images/km_operator.png"  >}}

{{< figure src="/static/images/km_operator.png"  >}}

In this view the supervisor when logged in gets a brief overview of the all the machines which are assigned to him from the shop floor. The image below shows the Supervisor Dashboard. The information such as  OEE, which part is being machined, Number of parts produced is show in realtime. Also the status of the images is colour coded (Red – Machining has been stopped, grey- Downtime and Green- Machining ). This allows the supervisor to have a quick look over the status of the machines. The fig 4.4 show the screenshot of the Dashboard when machining is being done The image is green to indicate machining progress

{{< figure src="/static/images/km_supervisor.png"  >}}

OEE shown in the image is of the machine for the day. Part running show the name of the Part which is being manufactured by that machines. Parts Produced show Number of parts produced by the operator

{{< figure src="/static/images/km_green.png"  >}}


The image below is a screenshot of the Scheduling Module of the System. Here supervisor will allot the parts that are to be manufactured to  operators. This info is automatically updated when operator logs in to his dashboard.

{{< figure src="/static/images/km_cycle.png"  >}}

The image below shows the Analysis module of the system. In this view the supervisor can see graphical representation of all the data collected from the machines in real-time. Graphs for Intracycle downtime, Setup time, reason for downtime etc. 


{{< figure src="/static/images/km_report.png"  >}}


Historical data can be viewed by selecting the time frame 

{{< figure src="/static/images/km_historical.png"  >}}


This project involves the design and development of IIoT hardware and supporting software for monitoring machine process by acquiring real-time data of Machine parameters such as Spindle, Clamp, Power and Alarm. The hardware is developed in such a way that it is plug and play. This avoids major overhaul of Legacy machines for Upgrading to IIoT. It is independent of CNC system. All the machines on the shop floor can be fitted with the system and tracked in real-time.
The Software developed is stored onto a central system which does the functions of storage and analysis of data. The Users with valid credentials only can access the data. The Integrated UI developed gives interactive environment to the user to monitor the machines on the shop floor


## Showcare

This project was presented at [IMTEX 2019 ](https://www.imtex.in/).

{{< figure src="/static/images/km_imtex.jpg"  >}}


