FROM golang:1.20.7 as builder

RUN apt-get update && apt-get install -y

WORKDIR /config
COPY . .

WORKDIR /go
RUN git clone https://github.com/multiversx/mx-chain-go.git && cd mx-chain-go && git checkout --force $(cat /config/binaryVersion)
WORKDIR /go/mx-chain-go

# MultiversX node
WORKDIR /go/mx-chain-go/cmd/node
RUN go build -v -buildmode=pie -ldflags="-X main.appVersion=$(git --git-dir /config/.git describe --tags --long --dirty)"
RUN cp /go/pkg/mod/github.com/multiversx/$(cat /go/mx-chain-go/go.mod | grep mx-chain-vm-v | sort -n | tail -n -1| awk -F '/' '{print$3}'| sed 's/ /@/g')/wasmer/libwasmer_linux_amd64.so /lib/libwasmer_linux_amd64.so
RUN cp /go/pkg/mod/github.com/multiversx/$(cat /go/mx-chain-go/go.mod | grep mx-chain-vm-go | sort -n | tail -n -1| awk -F '/' '{print$3}'| sed 's/ /@/g')/wasmer2/libvmexeccapi.so /lib/libvmexeccapi.so

WORKDIR /config
RUN cp -r * /go/mx-chain-go/cmd/node/config/

WORKDIR /go/mx-chain-go/cmd/node

# ===== SECOND STAGE ======
#FROM ubuntu:22.04 as ubuntu
#RUN apt-get update -y && apt-get upgrade -y

FROM scratch

#COPY --from=ubuntu "/lib" "/lib"
#COPY --from=ubuntu "/lib64" "/lib64"
COPY lib/ /lib
COPY lib64/ /lib64 
COPY --from=builder "/go/mx-chain-go/cmd/node" "/go/mx-chain-go/cmd/node/"
COPY --from=builder "/lib/libwasmer_linux_amd64.so" "/lib/libwasmer_linux_amd64.so"
COPY --from=builder "/lib/libvmexeccapi.so" "/lib/libvmexeccapi.so"
#COPY --from=ubuntu "/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2" "/lib64/"

WORKDIR /go/mx-chain-go/cmd/node/

EXPOSE 8080

ENTRYPOINT ["/go/mx-chain-go/cmd/node/node", "--rest-api-interface=:8080"]
