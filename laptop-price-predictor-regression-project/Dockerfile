# Base Image
FROM python:3.7

# WORK DIR
WORKDIR /app

COPY . /app

RUN pip install -r requirements.txt

EXPOSE 8501

CMD ["streamlit","run","app.py"]