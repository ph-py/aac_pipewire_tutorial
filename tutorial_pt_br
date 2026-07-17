### Tutorial: habilitar AAC para fones Bluetooth no Debian 13 (Trixie)

Este procedimento recompila o plugin Bluetooth do PipeWire com suporte a **AAC**, que vem desativado na compilação oficial do Debian por questões de dependência/licenciamento.

No caso deste tutorial, o resultado esperado é que o perfil abaixo apareça no GNOME/Pavucontrol:

```text
High Fidelity Playback (A2DP Sink, codec AAC)
```

> **Atenção:** este é um pacote local/customizado. Em futuras atualizações do `libspa-0.2-bluetooth`, será necessário repetir o processo para manter AAC.

#### 1. Confirme o cenário atual

Com o fone conectado, confira os codecs disponíveis:

```bash
pactl list cards | grep a2dp-sink
```

Antes da alteração, um caso típico mostra apenas:

```text
a2dp-sink: High Fidelity Playback (A2DP Sink, codec SBC)
a2dp-sink-sbc_xq: High Fidelity Playback (A2DP Sink, codec SBC-XQ)
```

Após o procedimento, deve aparecer também:

```text
a2dp-sink-aac: High Fidelity Playback (A2DP Sink, codec AAC)
```

Confira também a versão instalada do PipeWire:

```bash
apt policy pipewire libspa-0.2-bluetooth
```

Este tutorial foi usado com:

```text
Debian GNU/Linux 13 (trixie)
PipeWire 1.4.2-1
```

#### 2. Habilite os repositórios de código-fonte

Para baixar o código-fonte que corresponde aos pacotes instalados, é necessário ter entradas `deb-src`.

Abra o arquivo:

```bash
sudo nano /etc/apt/sources.list
```

Adicione estas linhas — ajuste somente se sua instalação usar outro espelho ou suite:

```text
deb-src http://deb.debian.org/debian/ trixie main contrib non-free non-free-firmware
deb-src http://security.debian.org/debian-security trixie-security main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ trixie-updates main contrib non-free non-free-firmware
```

Atualize o índice de pacotes:

```bash
sudo apt update
```

#### 3. Instale as ferramentas e dependências de compilação

```bash
sudo apt install build-essential devscripts
sudo apt build-dep pipewire
sudo apt install libfdk-aac-dev
```

A biblioteca `libfdk-aac-dev` é necessária **para compilar** o plugin com AAC.

A biblioteca de execução correspondente deve ser instalada automaticamente como dependência, mas confirme:

```bash
dpkg -s libfdk-aac2t64 | grep '^Status'
```

Resultado esperado:

```text
Status: install ok installed
```

#### 4. Baixe o código-fonte do PipeWire

Faça isso como seu usuário normal, **sem `sudo`**:

```bash
mkdir -p ~/pipewire-aac
cd ~/pipewire-aac
apt source pipewire
```

Entre no diretório de fonte criado. Para a versão usada neste caso:

```bash
cd ~/pipewire-aac/pipewire-1.4.2
```

Se o número de versão for diferente, use:

```bash
cd ~/pipewire-aac/pipewire-*
```

#### 5. Modifique a compilação para habilitar AAC

##### 5.1 Edite `debian/rules`

```bash
nano debian/rules
```

Localize:

```text
-Dbluez5-codec-aac=disabled
```

E troque para:

```text
-Dbluez5-codec-aac=enabled
```

Salve com `Ctrl+O`, confirme com `Enter` e saia com `Ctrl+X`.

##### 5.2 Edite `debian/control`

```bash
nano debian/control
```

Faça duas alterações:

1. Remova a linha que impede a compilação com a biblioteca AAC:

   ```text
   Build-Conflicts: libfdk-aac-dev
   ```

2. Na seção `Build-Depends:`, adicione:

   ```text
   libfdk-aac-dev,
   ```

Por exemplo, a dependência pode ficar próxima de `pkg-config`:

```text
Build-Depends: ...,
               pkg-config,
               libfdk-aac-dev,
               ...
```

Salve e feche o editor.

#### 6. Compile o pacote

Ainda dentro da pasta `pipewire-<versão>`, execute:

```bash
debuild -b -uc -us
```

Significado das opções:

- `-b`: gera pacotes binários;
- `-uc`: não assina o arquivo `.changes`;
- `-us`: não assina o pacote-fonte.

A compilação pode levar alguns minutos. Ao final, os arquivos `.deb` serão colocados na pasta acima, isto é, em `~/pipewire-aac`.

#### 7. Instale somente o plugin Bluetooth recompilado

Volte para a pasta onde os `.deb` foram gerados:

```bash
cd ~/pipewire-aac
```

Confira o arquivo criado:

```bash
ls -1 libspa-0.2-bluetooth_*.deb
```

Instale-o:

```bash
sudo dpkg -i libspa-0.2-bluetooth_*.deb
```

Se o `dpkg` reclamar de dependências, corrija com:

```bash
sudo apt --fix-broken install
```

E execute novamente a instalação:

```bash
sudo dpkg -i libspa-0.2-bluetooth_*.deb
```

#### 8. Reinicie o áudio e reconecte o fone

A opção mais simples e segura é reiniciar o computador:

```bash
sudo reboot
```

Alternativamente, reinicie os serviços de áudio do usuário:

```bash
systemctl --user restart wireplumber pipewire pipewire-pulse
```

Depois, desconecte e conecte o fone novamente.

#### 9. Verifique e selecione AAC

Confira se AAC apareceu:

```bash
pactl list cards | grep a2dp-sink
```

O resultado deve incluir:

```text
a2dp-sink-aac: High Fidelity Playback (A2DP Sink, codec AAC)
```

Para ativar por terminal, adapte o endereço Bluetooth ao seu dispositivo:

```bash
pactl set-card-profile bluez_card.F3_38_8C_32_01_E8 a2dp-sink-aac
```

Para descobrir o nome exato do card:

```bash
pactl list cards short
```

Você também pode selecionar pelo GNOME:

1. Abra **Configurações → Som**;
2. selecione seu fone Bluetooth;
3. escolha **High Fidelity Playback (A2DP Sink, codec AAC)**.

Ou pelo `pavucontrol`:

```bash
pavucontrol
```

Na aba **Configuração**, selecione o perfil AAC.

#### 10. Confirme o codec ativo

Use:

```bash
wpctl status
```

Encontre o ID do sink Bluetooth e inspecione-o. Exemplo:

```bash
wpctl inspect 106 | grep -iE 'codec|profile|bluez'
```

O resultado desejado é semelhante a:

```text
api.bluez5.codec = "aac"
api.bluez5.profile = "a2dp-sink"
```

### Evitando que uma atualização substitua o pacote

O repositório do Debian pode substituir seu `libspa-0.2-bluetooth` recompilado por uma versão oficial sem AAC. Para impedir isso temporariamente:

```bash
sudo apt-mark hold libspa-0.2-bluetooth
```

Confira os pacotes em hold:

```bash
apt-mark showhold
```

Para permitir atualização no futuro:

```bash
sudo apt-mark unhold libspa-0.2-bluetooth
```

> Antes de atualizar o PipeWire no futuro, é recomendável baixar a nova versão do fonte, reaplicar as duas alterações e recompilar o plugin AAC para a nova versão.

### O que manter instalado

Para AAC continuar funcionando, mantenha:

```text
libspa-0.2-bluetooth
libfdk-aac2t64
```

Também vale manter, embora sejam opcionais:

```text
pulseaudio-utils
pavucontrol
```

- `pulseaudio-utils` fornece o comando `pactl`, útil para verificar e trocar perfis.
- `pavucontrol` oferece uma interface gráfica mais detalhada para áudio e perfis Bluetooth.

### O que pode remover após concluir

Ferramentas usadas apenas para compilar podem ser removidas:

```bash
sudo apt remove libfdk-aac-dev devscripts build-essential
```

Antes de confirmar, faça uma simulação:

```bash
sudo apt remove --simulate libfdk-aac-dev devscripts build-essential
```

**Cancele** se o APT sugerir remover `libfdk-aac2t64` ou `libspa-0.2-bluetooth`.

Depois, você pode verificar dependências órfãs:

```bash
sudo apt autoremove --simulate
```

Se a lista fizer sentido:

```bash
sudo apt autoremove
```

### Preserve o pacote recompilado

Antes de apagar a pasta de compilação, guarde o `.deb` gerado:

```bash
mkdir -p ~/pacotes-personalizados
cp ~/pipewire-aac/libspa-0.2-bluetooth_*.deb ~/pacotes-personalizados/
```

Se precisar reinstalá-lo:

```bash
sudo dpkg -i ~/pacotes-personalizados/libspa-0.2-bluetooth_*.deb
```

A referência técnica para o procedimento original é o artigo [AAC and Debian, de Tookmund](https://tookmund.com/2024/02/aac-and-debian).
