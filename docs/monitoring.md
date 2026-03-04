Okay let’s build a monitoring and alert system, most of the work was done through the configuration in the GUI. Since it mainly involved setting up options step by step, I won’t go too deep into the process here and will just show the final results.
This is my Grafana dashboard. It helps me quickly check whether my application is still running properly. I used Blackbox Exporter to monitor the app’s availability, which is not a built-in metric in Prometheus. In addition, the dashboard also shows memory usage, CPU usage, and container restart rate. The design is not very fancy or beautiful, but it is simple and clear and most importantly, it works well for monitoring.
<img width="1920" height="1080" alt="image-2" src="https://github.com/user-attachments/assets/8a6752df-fdf3-4401-abfb-d383af909dd5" />



Besides monitoring, we also need an alert system. After all, we can’t stay in front of the screen 24/7, especially not while sleeping, right? For each application, I created three alert rules: CPU usage, memory usage, and probe status. When any of these metrics exceed the defined threshold, the system automatically sends a notification to my Telegram.

<img width="1833" height="793" alt="image" src="https://github.com/user-attachments/assets/95c94876-1b6e-4547-a75d-9db69cdab1ef" />


For example, if the Linkding project goes down at 1:25 AM, I immediately receive an alert message on Telegram. This feature is really helpful and gives me peace of mind. That said, I still want to improve the alert message content. Right now, it looks a bit complex and not very user-friendly, so that’s something I plan to refine next.

![834](https://github.com/user-attachments/assets/9e520efc-9569-4424-b22b-cb162e444dfe)
