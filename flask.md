Python file

```
from flask import Flask

app = Flask(__name__)


@app.route("/", methods=["GET"])
def home():
    return {"msg": "Hello"}


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=PORT)
```

Dockerfile

```
FROM python:3.11.4-alpine3.18

RUN apk update

RUN apk add --no-cache tzdata openssh curl
ENV TZ=Asia/Tashkent

WORKDIR /app

COPY requirements.txt requirements.txt

RUN pip install -r requirements.txt

COPY . .

CMD [ "python", "main.py" ]

```

requirements.txt

```
Flask==2.2.3
```