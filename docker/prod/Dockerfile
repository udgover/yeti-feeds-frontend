#  Container used to build the minified frontend JS code
FROM amd64/node:buster AS frontend-builder

RUN npm install -g n && n 16

ADD . /app
WORKDIR /app

RUN npm install
RUN npm run build

FROM nginx:latest AS frontend
COPY --from=frontend-builder /app/dist /www
EXPOSE 80
