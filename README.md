# Práctica de Configuración de Servidores DNS Maestro y Esclavo

## Introducción

Esta práctica consiste en la configuración de un servidor DNS maestro y un servidor DNS esclavo utilizando BIND (Berkeley Internet Name Domain). Se establecen zonas de dominio y se verifica la correcta resolución de nombres y la transferencia de zonas entre ambos servidores.

## Requisitos Previos

- Dos máquinas virtuales en una red privada:
  - **Servidor Maestro**: IP 192.168.57.103
  - **Servidor Esclavo**: IP 192.168.57.102

- Instalación de BIND en ambas máquinas:
  ```bash
  sudo apt update
  sudo apt install bind9

