FROM python:alpine
RUN pip install flask
ADD hello.py /
ENV FLASK_APP=/hello.py
CMD ["flask", "run", "-h", "0.0.0.0"]
