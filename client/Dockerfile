FROM tubbynl/grails as builder
WORKDIR /app
RUN grails create-app myreact -profile react
WORKDIR /app/myreact
RUN ./gradlew client:build
# CMD ["./gradlew", "client:start"]

FROM nginx
EXPOSE 3000
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/myreact/client/build /usr/share/nginx/html