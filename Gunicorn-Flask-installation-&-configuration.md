   **Install Flask and Gunicorn:**


1. Run command from python virtualenv:

pip install gunicorn flask

2. Create a file named app.py and paste below content (similar with cs-search) 


from flask import Flask
from flask_restful import Api
from resources.search import*

app = Flask(__name__)

api = Api(app)
#added routes for api
api.add_resource(Documents, '/corpus/v1/documents')
api.add_resource(DocumentDetails, '/corpus/v1/document_details')
if __name__ == "__main__":
    app.run("0.0.0.0", debug=True,threaded=True)

3. Create the WSGI Entry Point
We can simply import the Flask instance from our application and then run it:

Create new file wsgi.py with content:
from app import app

if __name__ == "__main__":
    app.run()
Note* (folder structure)
Search-engine-using-Elasticsearch-Jipa/
  |____ app.py
  |____ wsgi.py
  |____ services
  |____ ...
4. Testing Gunicorn’s Ability to Serve the Project
 
in Search-engine-using-Elasticsearch-Jipa/ execute 
gunicorn --bind 0.0.0.0:5000 wsgi:app

test in browser http://localhost:5000/

5. Create a systemd Unit File
Creating a systemd unit file will allow operating system init system to automatically start Gunicorn and serve our Flask application whenever the server boots.

Create a unit file ending in .service within the /etc/systemd/system
ex. cs-search.service
Inside, we’ll start with the [Unit] section, which is used to specify metadata and dependencies. We’ll put a description of our service here and tell the init system to only start this after the networking target has been reached:

[_Unit]_
_Description=Gunicorn instance to serve congnitive search cs-search module_
_After=network.target_
_# We will give our regular user account ownership of the process since it owns all of the relevant files_
_[Service]_
_# Service specify the user and group under which our process will run._
_User=jipag_
_# give group ownership to the root group so that Nginx can communicate easily with the Gunicorn processes._
_Group=nginx_
_# We'll then map out the working directory and set the PATH environmental variable so that the init system knows where our the executables for the process are located (within our virtual environment)._
_WorkingDirectory=/data2/corpus/Github/EM-Cognitivesearch/Search-engine-using-Elasticsearch-Jipa_
_Environment="PATH=/home/jipag/cognitive-search-prod/cognitive-search-prod/bin"_
_# We'll then specify the commanded to start the service_
_ExecStart=/home/jipag/cognitive-search-prod/cognitive-search-prod/bin/gunicorn --workers=1 --bind unix:cs-search.sock -m 000 wsgi:app_
_# This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running:_
_[Install]_
_WantedBy=multi-user.target_

_Note*_: In the last line of [Service] We tell it to start 3 worker processes. We will also tell it to create and bind to a Unix socket file within our project directory called app.sock. We’ll set a umask value of 007 so that the socket file is created giving access to the owner and group, while restricting other access. Finally, we need to pass in the WSGI entry point file name and the Python callable within.

We can now start, stop and status the Gunicorn service we created and enable it so that it starts at boot:
- sudo systemctl enable cs-search.service
- sudo systemctl start cs-search.service
- sudo systemctl stop cs-search.service
- sudo systemctl status cs-search.service

6. We’ll need to tell NGINX about our app and how to serve it.

Our Gunicorn application server should now be up and running, waiting for requests on the socket file in the project directory. We need to configure Nginx to pass web requests to that socket by making some small additions to its configuration file.
 - create empty file in app path cs-search.sock (user permissions are 777 for an user from nginx group)
   srwxrwxrwx  1 jipag nginx    0 Dec 19 19:05 cs-search.sock

- We’ll need to tell NGINX about our app and how to serve it.
   cd into /etc/nginx/
   we need to add is a location block that matches every request. Within this block, we’ll include the proxy_params file    that specifies some general proxying parameters that need to be set. We’ll then pass the requests to the socket we defined using the proxy_pass directive.
  sudo nano nginx.conf

added under server location:

        location /corpus/ {
                proxy_pass http://unix:/data2/corpus/Github/EM-Cognitivesearch/Search-engine-using-Elasticsearch-Jipa/cs-search.sock:/corpus/;
        }

or added as a single service (like a dev environment) under server location:

      location /corpus/ {
                proxy_pass http://localhost:8085/corpus/;
        }
- verify syntax error:
  sudo nginx -t

- restart nginx
  sudo systemctl restart nginx

   








