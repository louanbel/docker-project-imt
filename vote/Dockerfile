FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN apt-get update
RUN apt-get install -y --no-install-recommends curl
RUN pip3 install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
