FROM node:20
COPY . /usr/project
WORKDIR /usr/project
# RUN npm install --global yarn
RUN yarn
CMD [ "yarn", "run", "dev", "--port 80"]