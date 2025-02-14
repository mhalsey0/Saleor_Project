FROM python:3.12

#install system dependencies
RUN apt-get update && apt-get install -y \
    gettext \
    libffi8 \
    libgdk-pixbuf2.0-0 \
    liblcms2-2 \
    libopenjp2-7 \
    libssl3 \
    libtiff6 \
    libwebp7 \
    shared-mime-info \
    mime-support \
    libpq5

#clean up to reduce image size. 
RUN rm -rf /var/lib/apt/lists/*

#add project files
WORKDIR /app
COPY . .

#install python dependencies
RUN pip install --no-cache-dir poetry==1.8.4
RUN poetry config virtualenvs.create false
RUN poetry install --no-root


#create new group non-root user
RUN groupadd saleor_group && \
    useradd --create-home --shell /bin/bash -g saleor_group saleor

#assign ownship of directory to saleor user
RUN chown -R saleor:saleor_group /app

#configure static files
ARG STATIC_URL
ENV STATIC_URL=${STATIC_URL:-/static/}
RUN SECRET_KEY=dummy STATIC_URL=${STATIC_URL} python3 manage.py collectstatic --no-input

#expose port and start
EXPOSE 8000

CMD ["gunicorn", "--bind", ":8000", "--workers", "4", "--worker-class", "saleor.asgi.gunicorn_worker.UvicornWorker", "saleor.asgi:application"]