#Alpine da python base image seç
FROM python:3.7-alpine 
#/code kalsörüne git
WORKDIR /code
# export FLASK_APP=aap.py ile environment oluşturur
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
# Kurulması gerekli bir paker
RUN apk add --no-cache gcc musl-dev linux-headers
# requirenments.txt yi /code içine kopyalar
COPY requirements.txt requirements.txt
# python app için gerekli Flask ve Redis i kurar
RUN pip install -r requirements.txt
# Hostun 5000 portundan yayımla
EXPOSE 5000
# dizindeki herşeyi /cmd ye kopyala
# çalıştığım dizindeki doyaları /code içine kopyala
COPY . .
# flask run ile uygulamayı çalıştır.
CMD ["flask", "run"]