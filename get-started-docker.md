# Get Started with Docker

Let me install docker first

	sudo apt-get install docker.io 
	
Now I can run a nginx container with

	docker run -d nginx:1.10
	
Let me try to access, as if I installed nginx as native service. like 

	curl http://127.0.0.1

This does not work. Hell, why?

> Because the container is running in a box, completely isolated

I can add a port forward let the container communicate with outside world.

	docker run -d -p 8080:80 nginx:1.10

Now I can try 

	curl http://127.0.0.1:8080
	
If I have another site wanna to serve. I can just start another container

	docker run -d -p 8081:80 nginx:1.10	

And these two containers are isolated. 

> We can just configure one without even thinking about the other