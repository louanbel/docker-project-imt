FROM python:slim

WORKDIR /app

RUN apt-get update && apt-get install -y apache2-utils

COPY make-data.py .

RUN python make-data.py

COPY generate-votes.sh .

EXPOSE 5000

CMD ["bash", "generate-votes.sh"]
