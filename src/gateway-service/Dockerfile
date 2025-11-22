FROM python:3.10-slim-bullseye

RUN apt-get update && apt-get install -y --no-install-recommends --no-install-suggests build-essential libpq-dev python3-dev && pip install --no-cache-dir --upgrade pip

WORKDIR /app

# Create uploads folder (VERY IMPORTANT)
RUN mkdir -p /app/uploads

COPY ./requirements.txt /app
RUN pip install --no-cache-dir --requirement /app/requirements.txt

COPY . /app

EXPOSE 8080

CMD ["python", "server.py"]
