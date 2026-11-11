# Desbloquear a tela com a digital no Dell Latitude 3540 (Ubuntu + XFCE)

Passo a passo para usar o leitor de digital do **Dell Latitude 3540** no bloqueio de tela, em **Ubuntu 26.04** com **XFCE** (X11, LightDM, `xfce4-screensaver`).

## Identificar o leitor

```bash
lsusb | grep -i -E "finger|goodix|synaptics|elan|validity"
```

No Latitude 3540:

```
ID 27c6:63cc Shenzhen Goodix Technology Co.,Ltd. Goodix Fingerprint USB Device
```

Esse modelo é suportado pelo driver `goodixmoc` do libfprint que vem no Ubuntu 26.04. Para conferir outro leitor, procure o ID na [lista de dispositivos suportados do libfprint](https://fprint.freedesktop.org/supported-devices.html).

## Passo 1 — instalar

```bash
sudo apt install fprintd libpam-fprintd
```

Conferir se o leitor foi reconhecido:

```bash
fprintd-list "$USER"
```

## Passo 2 — cadastrar a digital

```bash
fprintd-enroll                        # indicador direito (padrão)
fprintd-enroll -f left-index-finger   # outro dedo
```

Encoste e levante o dedo várias vezes, variando um pouco a posição, até aparecer `enroll-completed`.

Testar a leitura:

```bash
fprintd-verify    # deve terminar com verify-match
```

## Passo 3 — ativar só no bloqueio de tela

O bloqueio de tela do XFCE é o `xfce4-screensaver`, que autentica por `/etc/pam.d/xfce4-screensaver`. Basta adicionar uma linha no topo desse arquivo:

```bash
sudo cp -a /etc/pam.d/xfce4-screensaver /etc/pam.d/xfce4-screensaver.bak-sem-digital
sudo sed -i '1i auth sufficient pam_fprintd.so max-tries=3 timeout=10' /etc/pam.d/xfce4-screensaver
```

O arquivo fica assim:

```
auth sufficient pam_fprintd.so max-tries=3 timeout=10
auth include common-auth
auth optional pam_gnome_keyring.so
```

- `sufficient`: se a digital conferir, desbloqueia; se não, segue para a senha.
- `max-tries=3 timeout=10`: espera a digital por até 3 tentativas ou 10 segundos antes de cair para a senha.

Login e `sudo` continuam só com senha. Para ligar a digital em tudo de uma vez (bloqueio, login e `sudo`), a alternativa é `sudo pam-auth-update` e marcar *Fingerprint authentication*.

## Testar

```bash
xflock4
```

Mexa o mouse e encoste o dedo no sensor.

## Limitação do XFCE

A tela de bloqueio não aceita digital e senha ao mesmo tempo: ela espera a digital primeiro e só libera o campo de senha depois das tentativas ou do tempo limite. Ajuste `timeout` se 10 segundos for muito.

## Desfazer

```bash
sudo mv /etc/pam.d/xfce4-screensaver.bak-sem-digital /etc/pam.d/xfce4-screensaver
fprintd-delete "$USER"    # apaga as digitais cadastradas
```

## Resultado

Dois dedos cadastrados (indicadores esquerdo e direito) e desbloqueio da tela por digital funcionando, com a senha como alternativa.
