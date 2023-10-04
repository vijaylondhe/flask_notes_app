# flask_notes_app

Solution
Log in to the server using the credentials provided:

ssh cloud_user@<PUBLIC_IP_ADDRESS>
Create the Build Files

Change to the notes directory:
cd notes
List the files in the directory:
ls -la
Inspect the config.py file:
cat config.py
Crate the .dockerignore file:
vim .dockerignore
In the file, paste the following:
.dockerignore
Dockerfile
.gitignore
Pipfile.lock
migrations/
Save the file:
ESC
:wq

Create the Dockerfile:
vim Dockerfile
In the file, paste the following:
FROM python:3
ENV PYBASE /pybase
ENV PYTHONUSERBASE $PYBASE
ENV PATH $PYBASE/bin:$PATH
RUN pip install pipenv

WORKDIR /tmp
COPY Pipfile .
RUN pipenv lock
RUN PIP_USER=1 PIP_IGNORE_INSTALLED=1 pipenv install -d --system --ignore-pipfile

COPY . /app/notes
WORKDIR /app/notes
EXPOSE 80
CMD ["flask", "run", "--port=80", "--host=0.0.0.0"]


Build and Setup Environment

Build the notesapp image:
docker build -t notesapp:0.1 .
Check the status of the image:
docker images
List the containers:
docker ps -a
List the docker networks:
docker network ls
Run a container using the notesapp image and mount the migrations directory:
docker run --rm -it --network notes -v /home/cloud_user/notes/migrations:/app/notes/migrations notesapp:0.1 bash
Once connected to the container, enable SQLAlchemy:
flask db init
Check the migrations folder:
ls -l migrations
Create the files needed to configure the database:
flask db migrate
Apply the files:
flask db upgrade
Run, Evaluate, and Upgrade

Run a container using the notesapp:0.1 image:
docker run --rm -it --network notes -p 80:80 notesapp:0.1

Using a web browser, navigate to the public IP address for the server.
Sign up for a new account using an email address and password.
Once you are signed up, log in to your account.
Create your first note.
Verify that you can edit the note.

Back in the terminal, disable Debug mode by editing the .env file:
vim .env
Remove the export FLASK_ENV='development' line.
Save the file:
ESC
:wq


Build the image again:
docker build -t notesapp:0.2 .
Run a container using the updated image:
docker run --rm -it --network notes -p 80:80 notesapp:0.2
In a web browser, navigate to the public IP address for the server, and log in to your account.
Verify that you can add a second note.
In the terminal, stop the container:
CTRL+C
Upgrade to Gunicorn

Check the Pipfile:
cat Pipfile
Run a container and modify the pip file:
docker run --rm -it -v /home/cloud_user/notes/Pipfile:/tmp/Pipfile notesapp:0.2 bash
Once connected, change to the /tmp directory:
cd /tmp
Add gunicorn to the list of dependencies:
pipenv install gunicorn
Exit the container:
exit

Verify that gunicorn was added under [packages]:
cat Pipfile

Modify the init.py script:
vim __init__.py
Beneath the import section, add the following:
from dotenv import load_dotenv, find_dotenv
load_dotenv(find_dotenv())
Save the file:
ESC
:wq

Open the Dockerfile:
vim Dockerfile
To the bottom of the file, make the following changes:
COPY . /app/notes
WORKDIR /app
EXPOSE 80
CMD ["gunicorn", "-b 0.0.0.0:80", "notes:create_app()"]
Save the file:
ESC
:wq
Build a Production Image

Build the updated notesapp image:
docker build -t notesapp:0.3 .
Run a detached container using the updated image:
docker run -d --name notesapp --network notes -p 80:80 notesapp:0.3
Check the status of the container:
docker ps -a
In a web browser, navigate to the public IP address for the server, and log in to your account.
Verify that you can create a new note.
