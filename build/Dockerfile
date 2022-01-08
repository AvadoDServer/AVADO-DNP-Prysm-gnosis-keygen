
FROM node:10.15.3 as build-deps-wizard

# build wizard
WORKDIR /usr/src/app/wizard
COPY wizard .
RUN yarn
RUN rm -Rf build && yarn run build

# get source code for validator-data-generator
WORKDIR /app
RUN git clone https://github.com/gnosischain/validator-data-generator.git

# WORKDIR /app
# RUN wget https://github.com/ethereum/eth2.0-deposit-cli/releases/download/v1.0.0/eth2deposit-cli-9310de0-linux-amd64.tar.gz
# RUN tar zxvf *.tar.gz
# RUN ls -l
# RUN rm *.tar.gz
# RUN mv */deposit ./
# RUN ls -l


# build final image
FROM python:3.7-alpine

# build validator-data-generator
WORKDIR /
COPY --from=build-deps-wizard /app/validator-data-generator/requirements.txt ./
COPY --from=build-deps-wizard /app/validator-data-generator/setup.py ./
COPY --from=build-deps-wizard /app/validator-data-generator/eth2deposit ./eth2deposit
RUN apk add --no-cache --update gcc libc-dev linux-headers libressl ca-certificates supervisor expect wget nginx nodejs npm docker
RUN pip3 install -r requirements.txt
RUN python3 setup.py install
RUN mkdir -p /app/validator_keys

#WORKDIR /usr/src/app
# RUN apk add --no-cache libressl ca-certificates supervisor expect wget nginx nodejs npm docker
# COPY --from=ghcr.io/gnosischain/validator-data-generator:latest /app/* /usr/src/app 
# RUN echo "APP="
# RUN ls -lR /usr/src/app
# RUN echo "----"
# #bin/deposit
# # COPY --from=build-deps-wizard /app/deposit /usr/bin/deposit
# RUN chmod +x /usr/bin/deposit


RUN mkdir -p /usr/src/app/scripts
COPY files/mkkeys.sh /usr/src/app/scripts
RUN chmod +x /usr/src/app/scripts/mkkeys.sh

# Wizard
COPY --from=build-deps-wizard /usr/src/app/wizard/build /usr/local/wizard

# Monitor
WORKDIR /usr/src/monitor
COPY monitor .
# COPY --from=build-deps-wizard /usr/src/app/monitor /usr/src/monitor
# WORKDIR /usr/src/monitor
RUN npm i --production

COPY files/entrypoint.sh /usr/src/app/scripts/entrypoint.sh
RUN chmod +x /usr/src/app/scripts/entrypoint.sh
COPY files/supervisord.conf /etc/supervisord/
COPY files/nginx.conf /etc/nginx/


ENTRYPOINT /usr/src/app/scripts/entrypoint.sh
