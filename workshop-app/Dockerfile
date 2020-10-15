FROM node:14.5.0-alpine As builder

WORKDIR /usr/src/app

COPY package.json package-lock.json ./

RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh

RUN npm install

COPY . .

RUN npm run build --prod

FROM nginx:1.15.8-alpine

COPY --from=builder /usr/src/app/dist/workshop-app/ /usr/share/nginx/html
