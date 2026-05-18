# Android HomeLab 🚀

Este repositório contém a documentação técnica, mapeamento de arquitetura e arquivos de configuração do meu ecossistema de HomeLab distribuído e híbrido. O projeto visa maximizar a eficiência energética e reaproveitar hardware legado, utilizando um smartphone antigo como servidor de alta disponibilidade (24/7) integrado a um nó principal de performance sob demanda.

---

## 🎯 Objetivo do Projeto

O principal objetivo deste projeto é a sustentabilidade tecnológica através da reciclagem de hardware (**Tech Recycling**). Em vez de descartar um dispositivo móvel antigo, ele foi transformado em um servidor de infraestrutura estável para serviços essenciais que exigem disponibilidade contínua, mitigando o consumo elétrico de manter um computador desktop ligado ininterruptamente. 

Adicionalmente, o projeto visa o estudo prático e aprofundado de:
* **Sistemas Operacionais Baseados em Linux** (Ambientes rolling release e ambientes restritos em arquitetura ARM).
* **Redes e VPNs Mesh** (Interconexão de múltiplos nós sem exposição de portas em NAT residencial).
* **Segurança de Redes** (Bloqueio de ameaças e telemetria a nível de DNS).
* **Orquestração e Automação de Infraestrutura** (Sistemas sob demanda acionados remotamente).

---

## 🛠️ Documentação do Ecossistema

Este laboratório opera sob uma arquitetura distribuída, onde cada serviço foi alocado no nó ideal de acordo com restrições de hardware, consumo de energia e capacidade de processamento.

### 1. Visão Geral da Arquitetura

```
                  [ Internet / Rede Externa ]
                              |
                              | (Tailscale VPN Mesh)
                              v
        +-------------------------------------+
        |                                     |
        v                                     v
+-----------------------+             +-----------------------+
|  Nó de Disponibilidade|             |   Nó de Performance   |
|        (24/7)         |             |     (Sob Demanda)     |
|                       |             |                       |
|   Samsung J7 Prime    |             |     PC Principal      |
|  (LineageOS / Debian) |             |     (Arch Linux)      |
|                       |             |                       |
|  - AdGuard Home       |== (WoL) ===>|  - Sunshine Server    |
|  - Navidrome          |             |  - Jellyfin(Docker)   |
|  - FileBrowser        |             |  - Portainer(Docker)  |
|                       |             |  - Filebrowser(Docker)|
+-----------------------+             +-----------------------+
        |                                     |
        +------------------+------------------+
                           |
                           v
              [ Flame Dashboard Central ]
```
---

### 2. Especificação dos Nós

#### 📱 Nó de Baixo Consumo (Servidor 24/7)
* **Dispositivo:** Samsung Galaxy J7 Prime
* **Sistema Operacional Host:** LineageOS 18.1 Unofficial (Android 11) com acesso root via Magisk.
* **Ambiente de Servidor:** Debian Linux rodando via Termux (`proot-distro`).
* **Camada de Rede:** Integrado à malha privada via cliente Tailscale nativo.
* **Serviços Hospedados:**
    * **AdGuard Home:** Atua como o servidor DNS principal da rede local, filtrando anúncios e rastreadores. *Nota de implementação:* Devido às restrições do kernel do Android para bindar portas abaixo de 1024, foi necessária uma regra de `prerouting` via `iptables` redirecionando o tráfego da porta padrão de DNS (53) para uma porta alta gerenciada pelo serviço.
    * **Navidrome:** Servidor de streaming de música focado em performance. Lida perfeitamente com transmissão de áudio em arquitetura ARM sem impactar o consumo de RAM do dispositivo.
    * **FileBrowser:** Gerenciador de arquivos leve e eficiente para acesso rápido ao armazenamento interno do dispositivo via interface web.

#### 💻 Nó de Alta Performance (Processamento sob Demanda)
* **Hardware Principal:** Processador AMD Ryzen 5 5600GT
* **Sistema Operacional:** Arch Linux com Hyprland
* **Ambiente de Servidor:** Docker Engine + Portainer para gerenciamento de containers.
* **Serviços Hospedados:**
    * **Sunshine:** Host de streaming de tela auto-hospedado de baixíssima latência para jogos e produtividade.
    * **Jellyfin Media Server:** Container Docker responsável por gerenciar e catalogar bibliotecas de vídeo pesadas (com suporte a sideload nativo na Smart TV).
    * **Portainer:** Interface gráfica de monitoramento e gerenciamento do ciclo de vida dos containers.
    * **Filebrowser:** Utilizado para acessar os arquivos do pc

---

### 3. Mecanismos de Integração e Automação

#### 🔄 Inicialização e Segurança da Sessão (Sunshine + WoL)
Para garantir que o Nó de Performance permaneça desligado quando não está em uso, a inicialização é controlada remotamente pelo J7 Prime através do protocolo **Wake-on-Lan (WoL)**, disparando o pacote mágico através do túnel criptografado da Tailscale.

Ao ligar, a integração de segurança funciona da seguinte forma:
1.  O Arch Linux realiza o **Autologin** automático na tty.
2.  O gerenciador de janelas (Hyprland) inicializa, permitindo que o Sunshine capture corretamente o display do servidor Wayland para o streaming.
3.  Simultaneamente, um script de pós-inicialização dispara o bloqueador de tela (`hyprlock`) imediatamente no primeiro milissegundo do boot. 
4.  **Resultado:** A sessão de streaming via Moonlight (no dispositivo cliente) fica totalmente operacional, enquanto o monitor físico do PC permanece bloqueado com segurança contra acessos locais não autorizados.

#### 🧭 Painel Centralizado (Flame Dashboard)
Toda a infraestrutura distribuída é unificada através do **Flame Dashboard**. Ele serve como a página inicial padrão em todos os dispositivos da rede local e da VPN, organizando de forma limpa os acessos aos serviços divididos por sua categoria de execução (Serviços 24/7 no J7 vs. Serviços sob demanda no PC).

---

## 📈 Considerações Finais de Infraestrutura

Este repositório não foca no processo exaustivo de instalação de Custom ROMs ou desbloqueio de bootloaders (visto que cada dispositivo possui ferramentas proprietárias como Odin, Heimdall, Fastboot ou SP Flash Tool). O foco deste espaço é puramente o reaproveitamento lógico do ecossistema e a otimização de redes.

Sinta-se à vontade para abrir uma *Issue* ou debater na aba de *Discussions* caso queira propor melhorias de proxy reverso, configurações de DNS ou otimizações de segurança!

---

## 📄 Licença

Este projeto está sob a licença **MIT**. Isso significa que você pode modificar, distribuir e usar a lógica desta arquitetura para fins pessoais ou comerciais, desde que inclua os devidos créditos.

*Isenção de responsabilidade: O autor não se responsabiliza por eventuais danos causados a dispositivos móveis (soft bricks, loops de boot ou desgaste de componentes) decorrentes de tentativas de modificação de sistema ou sobrecarga de hardware por parte de terceiros.*
