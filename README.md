# Opendistro for elasticsearch
Logs are a critical part of any system, they give you deep insights about your application, what your system is doing and what caused the error, when something wrong happens.Several times we came accross a situation where troubleshooting the application is required ,Debugging the error in the application across hundreds of log files on hundreds of servers can be very time consuming and complicated. A common approach to this problem is building a centralized logging application which can collect and aggregate different types of logs in one central location.There are many tools for this management many are the paid ones but we tried an approach that will be the boubust one also it should be the openSource.There are primarily four featues(funtionality) that we integrated in the present project.For that We have integrated Opendistro for elastic search with Logstash.
- Storing and collecting logs
 - Viuslizing them
 - Alerting them to the Slack Channel
 - Performance analysis of the cluster.
**We will going to look into these four parts and eventually see how we can build them in real world application environment.**
