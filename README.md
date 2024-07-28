# Minecraft Plugins - Home & Windplugin

Bem-vindo ao repositório dos plugins Home e Windplugin para Minecraft! Estes plugins permitem a personalização e funcionalidades adicionais para o servidor Minecraft, tornando a jogabilidade mais dinâmica e interativa.

## Índice

- [Sobre](#sobre)
- [Funcionalidades](#funcionalidades)
- [Configuração](#configuração)
- [Plugin Home](#plugin-home)
- [Plugin Windplugin](#plugin-Windplugin)
- [Estrutura do Projeto](#estrutura-do-projeto)

## Sobre

Este repositório contém dois plugins distintos para Minecraft: Home e Windplugin. Ambos são altamente configuráveis através do arquivo `config.yml` gerado após a inicialização do servidor, O plugin Home funciona utilizando um banco de dados MYSQL para as informações necessárias de uso.

## Funcionalidades
O plugin Home é utilizado pelos jogadores para marcar um local específico como um ponto fixo para teleporte em qualquer lugar do mapa, ao utilizar o comando "/sethome" o jogador define uma localização para qual ele irá se teleportar ao utilizar o comando "/home" o plugin tem um tempo de recarga (cooldown) ajustado de 60 segundos que pode ser alterado para outra quantidade desejável através do Config.yml gerado ao inicializar o server, o plugin também conta com uma característica visual de teleporte que pode ser alterada entre ligada ou desligada, nesta versão do código a partícula utilizada é  "PORTAL" do banco de dados do Bukkit.

O Plugin Windplugin é utilizado para configurar o projétil Wind Charge, garantindo um controle do tamanho da explosão gerada pelo projétil, velocidade de deslocamento do projétil e também adiciona partículas 

### Plugin Home

- **Comando `/sethome`**: Define a localização atual do jogador como sua "Home".
- **Comando `/home`**: Teleporta o jogador para sua "Home".
- **Configuração de Cooldown**: Tempo de espera configurável para usar o comando `/home`.
- **Partículas**: Ativação de partículas visuais para o teletransporte.

### Plugin Windplugin

- **Configuração da Explosão**: Poder da explosão configurável.
- **Velocidade do Projétil**: Velocidade do projétil ajustável.
- **Partículas de Explosão**: Ativação de partículas visuais na explosão.

## Configuração

Ambos os plugins podem ser configurados pelo arquivo `config.yml` gerado após a inicialização do servidor. Abaixo estão os detalhes das opções de configuração para cada plugin.

### Plugin Home
```yaml
# config.yml

cooldown: 60           # Tempo de cooldown (em segundos) para o uso do comando /home
particles: true        # Ativa/desativa partículas visuais no teletransporte
```
### Plugin Windplugin
```yaml
# config.yml

explosion-power: 8.0          # Poder da explosão, definido como 8.0 por padrão
projectile-speed: 1.0         # Velocidade do projétil, definida como 1.0 por padrão
enable-particles: true        # Ativa/desativa partículas visuais na explosão
```

### Estrutura e Lógica de implementação:

### Lógica do Plugin Home

O `HomeCommand` permite que os jogadores se teletransportem para uma localização "Home" previamente definida. Abaixo está uma visão geral da lógica de implementação:

1. **Verificação de Tipo de Remetente**:
   - Apenas jogadores podem usar o comando `/home`.

2. **Verificação de Cooldown**:
   - O comando verifica se o cooldown está ativo para o jogador. Se o cooldown não expirou, uma mensagem informa o tempo restante.

3. **Recuperação da Localização "Home"**:
   - A localização "Home" do jogador é recuperada do banco de dados de forma assíncrona.
   - Se uma localização é encontrada, o jogador é teleportado para essa localização, e partículas são geradas se configuradas.
   - Se nenhuma localização é encontrada, uma mensagem de erro é enviada.

4. **Tratamento de Erros**:
   - Se ocorrer um erro ao acessar o banco de dados, uma mensagem de erro é registrada e enviada ao jogador.

### Exemplo de Código
### /sethome
```java
public SetHomeCommand(HomePlugin plugin) {
        this.plugin = plugin;
    }

    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        if (!(sender instanceof Player)) {
            sender.sendMessage("Only players can use this command.");
            return true;
        }

        Player player = (Player) sender;
        String playerName = player.getName();
        double x = player.getLocation().getX();
        double y = player.getLocation().getY();
        double z = player.getLocation().getZ();
        float yaw = player.getLocation().getYaw();
        float pitch = player.getLocation().getPitch();

        try (Connection connection = plugin.getDatabaseManager().getConnection()) {
            PreparedStatement statement = connection.prepareStatement(
                    "REPLACE INTO homes (player, x, y, z, yaw, pitch) VALUES (?, ?, ?, ?, ?, ?)"
            );
            statement.setString(1, playerName);
            statement.setDouble(2, x);
            statement.setDouble(3, y);
            statement.setDouble(4, z);
            statement.setFloat(5, yaw);
            statement.setFloat(6, pitch);
            statement.executeUpdate();
            player.sendMessage("Home set successfully.");
        } catch (SQLException e) {
            player.sendMessage("An error occurred while setting your home.");
            plugin.getLogger().log(Level.SEVERE, "Could not save home location to the database.", e);
        }

        return true;
    }
}
```
### /home
```Java
public HomeCommand(HomePlugin plugin) {
        this.plugin = plugin;
    }

    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        if (!(sender instanceof Player)) {
            sender.sendMessage("Only players can use this command.");
            return true;
        }

        Player player = (Player) sender;
        UUID playerId = player.getUniqueId();
        long cooldownTime = plugin.getConfig().getLong("cooldown");

        if (cooldowns.containsKey(playerId)) {
            long lastUsed = cooldowns.get(playerId);
            long timeElapsed = (System.currentTimeMillis() - lastUsed) / 1000;

            if (timeElapsed < cooldownTime) {
                player.sendMessage("You must wait " + (cooldownTime - timeElapsed) + " seconds to use this command again.");
                return true;
            }
        }

        new BukkitRunnable() {
            @Override
            public void run() {
                try (Connection connection = plugin.getDatabaseManager().getConnection()) {
                    PreparedStatement statement = connection.prepareStatement(
                            "SELECT x, y, z, yaw, pitch FROM homes WHERE player = ?"
                    );
                    statement.setString(1, player.getName());
                    ResultSet resultSet = statement.executeQuery();

                    if (resultSet.next()) {
                        double x = resultSet.getDouble("x");
                        double y = resultSet.getDouble("y");
                        double z = resultSet.getDouble("z");
                        float yaw = resultSet.getFloat("yaw");
                        float pitch = resultSet.getFloat("pitch");

                        Location homeLocation = new Location(player.getWorld(), x, y, z, yaw, pitch);

                        Bukkit.getScheduler().runTask(plugin, () -> {
                            player.teleport(homeLocation);
                            if (plugin.getConfig().getBoolean("particles")) {
                                player.getWorld().spawnParticle(org.bukkit.Particle.PORTAL, homeLocation, 100);
                            }
                            player.sendMessage("Teleported to your home.");
                        });

                        cooldowns.put(playerId, System.currentTimeMillis());
                    } else {
                        Bukkit.getScheduler().runTask(plugin, () -> player.sendMessage("No home set."));
                    }
                } catch (SQLException e) {
                    plugin.getLogger().log(Level.SEVERE, "Could not retrieve home location from the database.", e);
                    Bukkit.getScheduler().runTask(plugin, () -> player.sendMessage("An error occurred while retrieving your home."));
                }
            }
        }.runTaskAsynchronously(plugin);

        return true;
    }
}
```
## Lógica de Implementação

### Lógica de Implementação do Windplugin

O `WindChargeListener` é responsável por adicionar funcionalidades ao item Wind Charge no Minecraft, permitindo que ele seja usado como um projétil explosivo com várias configurações personalizáveis. Abaixo está uma visão geral da lógica de implementação:

### WindChargeListener.java

#### Atributos Principais

- `plugin`: Referência à instância do plugin principal (`JavaPlugin`), permitindo acesso à configuração.
- `explosionPower`, `projectileSpeed`, `particleScale`: Configurações personalizáveis para o poder da explosão, velocidade do projétil e escala das partículas.
- `enableParticles`: Booleano que controla se as partículas de explosão são habilitadas.

#### Exemplo:

```java
  public void reloadConfig() {
        plugin.reloadConfig();
        FileConfiguration config = plugin.getConfig();
        this.explosionPower = config.getDouble("explosion-power", 8.0);
        this.projectileSpeed = config.getDouble("projectile-speed", 4.0);
        this.particleScale = config.getDouble("particle-scale", 1.0); // Default scale is 1.0
        plugin.getLogger().info("Config reloaded: Explosion power set to: " + this.explosionPower);
        plugin.getLogger().info("Config reloaded: Projectile speed set to: " + this.projectileSpeed);
        plugin.getLogger().info("Config reloaded: Particle scale set to: " + this.particleScale);
    }
```
