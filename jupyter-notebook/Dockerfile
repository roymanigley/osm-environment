FROM python:3.9-alpine

RUN apk add gcc python3-dev musl-dev linux-headers
RUN pip install pip --upgrade
RUN pip install jupyter pandas matplotlib seaborn requests 

RUN adduser -S jupyter
RUN mkdir /opt/jupyter
RUN chown jupyter /opt/jupyter -R

USER jupyter
WORKDIR /opt/jupyter

CMD jupyter notebook /opt/jupyter --ip 0.0.0.0 --allow-root --NotebookApp.token='' --NotebookApp.password=''
