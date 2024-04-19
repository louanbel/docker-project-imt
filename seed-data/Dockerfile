FROM python:3.9-slim

WORKDIR /app

RUN apt-get update && apt-get install -y apache2-utils

COPY make-data.py .

RUN python make-data.py

COPY generate-votes.sh .

CMD ["bash", "generate-votes.sh"]
