🚀 One Kubernetes concept that confused me when I started was the difference between containerPort, port, and targetPort.
At first, they all looked similar in YAML, but they actually serve different purposes.

📦 containerPort in Deployment
--------------------------------
 This simply tells Kubernetes which port the application listens on inside the container.
Many people think Kubernetes uses this value for routing traffic, but it doesn't. It's mainly there for documentation and clarity.

🌐 Service port
----------------
 This is the port that users or other applications connect to.
 Think of it as the front door of your Service.

 🎯 targetPort
 -----------------
 This is where the Service forwards the traffic inside the Pod.

 🎯 Golden Rule
 ----------------
 ✅ targetPort must match the port where the application is actually listening.

 If your application runs on port 8080 but your Service forwards traffic to 9090, requests will fail even though everything looks fine in Kubernetes.

🧠 Easy Way to Remember
📦 containerPort → Application listening port (informational)
🌐 port → Service entry point
🎯 targetPort → Actual destination port inside the Pod

Traffic Flow:
 👤 User → 🌐 Service Port → 🎯 targetPort → 📦 Application

🔥 Mastering these networking basics makes debugging Kubernetes Services much easier.
Sometimes the smallest Kubernetes concepts save the most troubleshooting time.

What's the Kubernetes concept that confused you the most when you first started?
👇 Share in the comments.

