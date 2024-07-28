# Minecraft
Plugin Home/Wind_Charge//
Ambos os Plugins podem ser configurados pelo arquivo Config.yml gerado após a inicialização do servidor.

Para o Plugin Home, ao utilizar o comando /sethome você grava o local onde o jogador está para ser a "Home", e ao utilizar /home você se teleporta para o local que foi gravado anteriormente.
Na área de config temos as opções de Cooldown e Particles, onde você pode alterar o tempo de cooldown para o uso do comando e ativar as partículas (Utilizei as partículas "PORTAL" do banco de dados do Bukkit por questões de estética)
cooldown: 60
particles: true
Como solicitado eu utilizei o MYSQL como banco de dados para o Plugin.

"Já o Plugin de Wind Charge foi configurado com as seguintes opções:
explosion-power: 8.0
projectile-speed: 1.0
enable-particles: true"

Ao configurar as partículas de explosão do WindCharge eu utilizei EXPLOSION_EMITTER do Bukkit, pois no banco de dados mais atualizados ainda não existia os frames específicos da explosão criada pelo WindCharge do Vanilla.

Muito Obrigado por ler.
