FROM python:3.12


#install system dependencies
RUN apt-get update && apt-get install -y \
    gettext \
    libffi8 \
    libgdkpixbuf2.00 \
    liblcms22 \
    libopenjp27 \
    libssl3 \
    libtiff6 \
    libwebp7 \
    sharedmimeinfo \
    mimesupport \
    libpq5

#clean up to reduce image size. 
RUN rm -rf /var/liv/apt/lists/*

#install python dependencies
RUN pip install "poetry==1.8.4"
WORKDIR /app
COPY pyproject.toml poetry.lock /app/
RUN poetry install --no-root && poetry config virtualenvs.create false

#create new group non-root user
RUN groupadd saleor_group && \
    useradd --create-home --shell /bin/bash -g saleor_group saleor

#assign ownship of directory to saleor user
RUN chown -r saleor:saleor /app

USER saleor

#configure static files
ARG STATIC_URL
ENV STATIC_URL=${STATIC_URL:-/static/}
RUN SECRET_KEY=dummy STATIC_URL=${STATIC_URL} python3 manage.py collectstatic --no-input

#expose port and start

EXPOSE 8000

CMD ["gunicorn", "--bind", ":8000", "--workers", "4", "--worker-class", "saleor.asgi.gunicorn_worker.UvicornWorker", "saleor.asgi:application"]