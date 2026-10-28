FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

ENV PYTHONUNBUFFERED=1
ENV REGNO=202401100200340
ENV APP_VERSION=dev

EXPOSE 5000

CMD ["python", "app.py"]