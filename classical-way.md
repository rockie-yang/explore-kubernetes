# If I install nginx with navtive OS tools


I can install like this if I am working ubuntu/debian
 
	sudo apt-get update
	sudo apt-get install nginx

I start the nginx service like

	sudo systemctl start nginx
	
I can check the service like 

	sudo systemctl status nginx
	
I can use the service with `curl` or in browser.

	curl http://127.0.0.1

So the question is how it really works?

> Becuase nginx open port 80 to serve http traffic

If I wanna serve more then one site on one OS?

> It is solvable by provide nginx more configuration
> 
> Why all configuration in one place, the configuration for one site interfere with the other.