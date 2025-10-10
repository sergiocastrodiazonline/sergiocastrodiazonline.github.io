---
title: "Writeup - Uploader | TheHackerLabs"
author: Sergi Castro
date: 2025-10-10
categories: [CTF, Pentesting, VulnHub, Privilege Escalation]
tags: [nmap, gobuster, reverse-shell, john, hashcat, unrestricted-file-upload, sudo, tar, privilege-escalation]
---

[Link Maquina](https://labs.thehackerslabs.com/machine/127)

## Contenido
- [Introduccion](#introduccion)
- [Objetivos](#objetivos)
- [Enumeracion de Servicios](#enumeracion-de-servicios)
- [Exploracion Web](#exploracion-web)
- [Enumeracion de Directorios con Gobuster](#enumeracion-de-directorios-con-gobuster)
- [Explotacion mediante Unrestricted File Upload](#explotacion-mediante-unrestricted-file-upload)
- [Enumeracion Post-Explotación](#enumeracion-post-explotacion)
- [Analisis del Zip](#analisis-del-zip)
- [Crackeo del Zip](#crackeo-del-zip)
- [Decodificar Hash](#decodificar-hash)
- [Acceso como operatorx](#acceso-como-operatorx)
- [Escalado a Root](#escalado-a-root)
- [Conclusiones](#conclusiones)
- [Resumen de Vulnerabilidades](#resumen-de-vulnerabilidades)

---





