
# Ambiente de testagem de aplicações Nodejs com Vagrant e Docker  


# 1 - Criei a imagem Docker de acordo com minha aplicação:

~~~docker
FROM node:16-alpine as builder

ENV NODE_ENV build

USER node
WORKDIR /home/node

COPY package*.json ./
RUN npm ci

COPY --chown=node:node . .
RUN npm run build \
    && npm prune --production
EXPOSE 3000:3000

# ---

FROM node:16-alpine

ENV NODE_ENV production

USER node
WORKDIR /home/node

COPY --from=builder --chown=node:node /home/node/package*.json ./
COPY --from=builder --chown=node:node /home/node/node_modules/ ./node_modules/
COPY --from=builder --chown=node:node /home/node/dist/ ./dist/

ENTRYPOINT ["node","dist/main"]

EXPOSE 3000:3000
~~~
### Nesse caso usei como base o node:16-alpine para fazer um pull da minha imagem no docker hub já que minha aplicação era Node.js
---

# 2 -Após montar a imagem, fiz um pull da mesma no meu repositório no Docker Hub:
<img align="center" alt="Exemplo docker_hub" height="600" width="1000" src="./images/Captura de tela de 2022-04-24 10-39-48.png">

---

# 3 - Depois criei o provisionamento em no Vagrant:
~~~Vagrant
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/xenial64"
    config.vm.network :forwarded_port, guest: 3000 , host: 3000
    config.vm.provision "docker" do |d|
            d.pull_images "aldry1303/despesa:1.0.2"
            config.vm.provision "shell", inline: "docker run -p 3000:3000 -d aldry1303/despesa"
    end
  end
~~~
- Note que após setar a imagem, foi necessário fazer a chamada do provisionamento do docker passando para ele a imagem e tag da nossa imagem.
- Na parte de Network temos que definir a forma que nossa máquina vai se comportar de acordo com a rede, no meu caso eu setei para ela uma porta guest(Que deve ser a mesma exposta no meu container) e uma porta host(Para poder encontrar meu container no ambiente).

# 4 - Agora é só um comandinho e correr para o abraço:

* Usei esse comando para iniciar uma máquina linux com sua imagem contenizada rodando dentro já exposta para um teste rápido.
~~~shell
vagrant up --provision
~~~