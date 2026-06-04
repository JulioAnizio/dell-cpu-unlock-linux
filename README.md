# 🔓 Dell CPU Unlock — Fix para processador travado em 500MHz no Linux

![Linux Mint](https://img.shields.io/badge/Linux%20Mint-87CF3E?style=for-the-badge&logo=linux-mint&logoColor=white)
![Dell](https://img.shields.io/badge/Dell-007DB8?style=for-the-badge&logo=dell&logoColor=white)
![Intel](https://img.shields.io/badge/Intel-0071C5?style=for-the-badge&logo=intel&logoColor=white)

## 🖥️ Equipamento testado

- **Modelo:** Dell Inspiron 5448
- **Processador:** Intel Core i5 5ª geração
- **RAM:** 8GB
- **Sistema:** Linux Mint
- **Fonte:** Genérica (não original Dell)

---

## ❌ O Problema

Ao usar uma fonte de alimentação genérica (não original Dell), a BIOS entende que o notebook está sem energia adequada e ativa o **BD PROCHOT** — um mecanismo de proteção que trava o processador em **~500MHz**, independentemente de qualquer configuração de energia do sistema operacional.

### Sintomas
- Máquina extremamente lenta, travando ao abrir 2 ou 3 janelas do Chrome
- `htop` mostrando CPU com uso de 100% em tarefas simples
- Comando `watch -n 1 "grep MHz /proc/cpuinfo"` revelando frequência de **~498-500MHz**
- Nenhuma configuração de governor (`performance`, `ondemand`) resolve o problema
- O problema **volta a cada reinicialização**

### Por que acontece?
A Dell usa um pino extra no conector da fonte para identificar se é original. Com fonte genérica, a BIOS recebe um sinal de "emergência" (BD PROCHOT — Bi-Directional Processor Hot) e limita o clock do processador para proteger o hardware. O sistema operacional não tem controle sobre isso pois o override vem direto do hardware.

---

## ✅ A Solução

Usar o pacote `msr-tools` para escrever diretamente nos **Model Specific Registers (MSR)** do processador, desativando o sinal de BD PROCHOT.

### Passo a passo

**1. Instalar as ferramentas necessárias**
```bash
sudo apt install msr-tools -y
```

**2. Carregar o módulo do kernel**
```bash
sudo modprobe msr
```

**3. Desativar o BD PROCHOT via registro MSR**
```bash
sudo wrmsr -a 0x1FC 2
```

**4. Verificar se funcionou**
```bash
watch -n 1 "grep MHz /proc/cpuinfo"
```
Os núcleos devem subir para **2500MHz ou mais** imediatamente.

---

## 🔁 Automatizando para cada inicialização

O comando `wrmsr` não persiste após reboot. Para aplicar automaticamente, crie um serviço systemd:

**1. Crie o script**
```bash
sudo nano /usr/local/bin/dell-cpu-unlock.sh
```

Conteúdo do script:
```bash
#!/bin/bash
modprobe msr
wrmsr -a 0x1FC 2
```

**2. Dê permissão de execução**
```bash
sudo chmod +x /usr/local/bin/dell-cpu-unlock.sh
```

**3. Crie o serviço systemd**
```bash
sudo nano /etc/systemd/system/dell-cpu-unlock.service
```

Conteúdo do serviço:
```ini
[Unit]
Description=Dell CPU Unlock - Disable BD PROCHOT
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/dell-cpu-unlock.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

**4. Ative o serviço**
```bash
sudo systemctl enable dell-cpu-unlock.service
sudo systemctl start dell-cpu-unlock.service
```

---

## 📊 Resultado

| | Antes | Depois |
|---|---|---|
| Frequência da CPU | ~500MHz | 2500–2700MHz |
| Uso da CPU (YouTube) | ~100% | 12–16% |
| RAM em uso | Irrelevante (CPU era o gargalo) | ~2.18GB de 8GB |
| Estabilidade | Travamentos constantes | Sistema estável |

---

## ⚠️ Aviso

Este fix bypassa uma proteção de hardware da BIOS da Dell. Funciona bem com fontes genéricas de boa qualidade, mas monitore a temperatura do notebook. Se a fonte não fornecer amperagem suficiente sob carga alta, o sistema pode apresentar instabilidade.

Para uso de desenvolvimento (VS Code, terminal, browser) funciona perfeitamente.

---

## 🤝 Contribuição

Este fix foi descoberto após 4 horas de diagnóstico com htop, terminal e bastante persistência. Se funcionou para você, deixa uma ⭐ no repositório!

Testado em **Dell Inspiron 5448** com **Linux Mint**. Pode funcionar em outros modelos Dell com o mesmo problema.

---

*Solução documentada por [Julio](https://github.com/julioanizio) — Dev em transição de carreira, Linux enthusiast e sobrevivente da fonte genérica Dell* 😄
