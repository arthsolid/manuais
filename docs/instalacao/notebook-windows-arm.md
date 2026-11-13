# Instalação do ERP em notebook Windows ARM

*Manual para técnicos. Última revisão: 12/11/2026.*

## Quando usar este manual

Use quando for instalar o sistema num notebook novo e ele tiver processador ARM (Qualcomm Snapdragon). Esses notebooks rodam o ERP e o Oracle "emulados", ou seja, o Windows traduz o programa para funcionar no processador, e isso causa alguns erros que não aparecem num PC comum (Intel/AMD).

**Como saber se o notebook é ARM:** no Windows, pesquise por "variáveis de ambiente" e abra **Editar as variáveis de ambiente do sistema** > botão **Variáveis de Ambiente** > em "Variáveis do sistema", procure **PROCESSOR_ARCHITECTURE**. Se estiver **ARM64**, é ARM.

## Antes de começar

- **IP do servidor da loja:** num PC que já entra no sistema, clique com o botão direito no atalho do **iSOLIDUS** > **Abrir local do arquivo**. O endereço aparece na barra do Explorador, por exemplo `\\192.168.0.10\Solidus`. O IP é a parte `\\192.168.0.10`.
- **Usuário e senha** da pasta de rede e o login de teste do sistema: pegue com a equipe (não ficam neste manual).

## Passo 1. Entrar na pasta do servidor

1.  No notebook, abra o Explorador de Arquivos, digite `\\IP` na barra de endereço (ex.: `\\192.168.0.10`) e aperte Enter.
2.  Devem aparecer as pastas **Solidus** e **Temp**.

**Se o Windows disser que não localiza o caminho:** abra o Prompt de Comando e rode `net use \\IP\Solidus /user:IP\administrador`. Se der **Erro de sistema 3227320323**, a rede está funcionando: quem bloqueia é o Windows 11, que não aceita a pasta compartilhada do servidor sem "assinatura". Para liberar:

1.  Aperte Win+R, digite `regedit` e Enter.
2.  Vá até `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters`.
3.  Deixe `AllowInsecureGuestAuth` com valor **1** (se não existir, crie: botão direito > Novo > Valor DWORD (32 bits)).
4.  Crie também o valor DWORD (32 bits) `RequireSecuritySignature` com valor **0**. Foi esse que destravou no caso real.
5.  Reinicie o notebook e teste o `\\IP` de novo.

Em Windows Pro dá para fazer o mesmo pelo `gpedit.msc` > Configuração do Computador > Modelos Administrativos > Rede > Estação de Trabalho do Lanman > **Habilitar logons de convidado não seguros** = Habilitado. No Windows Home não existe gpedit, por isso o caminho é o registro.

## Passo 2. Instalar o Oracle

1.  Dentro de `\\IP\Temp`, copie a pasta `client` para o `C:`. Se só tiver o zip `win32_11gR2_client`, copie e descompacte (dá na mesma).
2.  Abra o `C:\client\setup.exe`.
3.  Tipo de instalação: **Administrador**.
4.  Oracle Base: `C:\oracle\app`
5.  Local do software: `C:\oracle\app\product\11.2.0\client_1` (confira com atenção, esse caminho é o que mais dá problema).
6.  Copie o `tnsnames.ora` da `\\IP\Temp` e cole em `C:\oracle\app\product\11.2.0\client_1\network\admin`.

## Passo 3. Conferir a variável ORACLE_HOME

É ela que diz ao Windows onde o Oracle está. No caso real ela estava errada (sem o `app` no meio) e o Oracle não funcionava.

1.  Abra as Variáveis de Ambiente (mesmo caminho do "Como saber se é ARM").
2.  Em "Variáveis do sistema", procure **ORACLE_HOME**. Tem que ser exatamente `C:\oracle\app\product\11.2.0\client_1`. Se estiver diferente, clique em **Editar** e corrija.
3.  Para testar: abra um Prompt de Comando novo e rode `tnsping NOME_DO_BANCO` (o nome que está no tnsnames.ora, ex.: `orcl`). Tem que terminar em **OK**. Se aparecer "Message 3511 not found" ou "TNS-03505", a ORACLE_HOME ainda está errada.

## Passo 4. Atalhos e primeira abertura

1.  Na Área de Trabalho: botão direito > **Novo** > **Atalho** > cole `\\IP\Solidus\iSOLIDUS.exe` > Avançar > nome **iSOLIDUS** > Concluir.
2.  Dê dois cliques no iSOLIDUS. Ele copia o sistema do servidor e cria a pasta `C:\Solidus` no notebook.
3.  Depois disso, botão direito em `C:\Solidus\SOL_ERP.exe` > **Enviar para** > **Área de trabalho (criar atalho)**.

## Passo 5. Ajustar a emulação do sistema (específico do ARM)

**Sintoma:** mesmo com rede, Oracle e tnsping OK, o sistema mostra "Não foi possível conectar no banco de dados" e "Missing DriverName property."

1.  Botão direito em `C:\Solidus\SOL_ERP.exe` > **Propriedades** > guia **Compatibilidade**.
2.  Clique em **Alterar configurações de emulação**.
3.  Troque de "Padrão" para um modo mais compatível (Seguro ou Estrito) e clique em OK.
4.  Abra o sistema de novo. No caso real, o erro mudou para o do Passo 6.

## Passo 6. Erro "No mapping for the Unicode character..."

**Sintoma:** ao abrir aparece "No mapping for the Unicode character exists in the target multi-byte code page."

1.  Feche o sistema.
2.  Apague o arquivo `C:\Solidus\licsol_erp.json`.
3.  Abra o sistema **só pelo atalho do iSOLIDUS**. No caso real, a partir daqui o sistema abriu normalmente.

## Passo 7. Testar

Entre com o login de teste (pegar com a equipe). Abriu normalmente = instalação pronta.

## Se ainda não funcionar

- Teste a rede até o banco: no PowerShell, `Test-NetConnection IP -Port 1521` tem que dar **True**.
- Abra o `tnsnames.ora` no Bloco de Notas e confira o IP do servidor (HOST) e a porta (PORT, normalmente 1521).
- Copiar o arquivo SCONF de `\\IP\Solidus` para `C:\Solidus` é uma dica comum para o "Missing DriverName", mas no notebook ARM sozinho não resolveu.
- Se nada der certo, a saída é usar outra máquina (Intel/AMD) ou acessar o sistema por área de trabalho remota.

*Baseado num atendimento real de 05/11 a 11/11/2026.*
