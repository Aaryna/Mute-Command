# Mute-Command
package com.example.muteplugin;

import org.bukkit.ChatColor;
import org.bukkit.command.Command;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.player.AsyncPlayerChatEvent;
import org.bukkit.plugin.java.JavaPlugin;

import java.util.HashMap;
import java.util.Map;

public class MutePlugin extends JavaPlugin implements Listener {

// Map to store muted players and their mute durations
private final Map<Player, Long> mutedPlayers = new HashMap<>();

@Override
public void onEnable() {
    // Register events
    getServer().getPluginManager().registerEvents(this, this);
}

@Override
public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
    if (command.getName().equalsIgnoreCase("mute")) {
        // Check for required arguments
        if (args.length < 2) {
            sender.sendMessage(ChatColor.RED + "Usage: /mute <player> <duration>");
            return true;
        }

        // Parse player argument
        Player target = getServer().getPlayer(args[0]);
        if (target == null) {
            sender.sendMessage(ChatColor.RED + "Player not found.");
            return true;
        }

        // Parse duration argument
        long duration = parseDuration(args[1]);
        if (duration <= 0) {
            sender.sendMessage(ChatColor.RED + "Invalid duration.");
            return true;
        }

        // Mute the player
        mutePlayer(target, duration);

        // Notify the sender
        sender.sendMessage(ChatColor.GREEN + "Muted player " + target.getName() + " for " + formatDuration(duration) + ".");

        return true;
    } else if (command.getName().equalsIgnoreCase("unmute")) {
        // Check for required arguments
        if (args.length < 1) {
            sender.sendMessage(ChatColor.RED + "Usage: /unmute <player>");
            return true;
        }

        // Parse player argument
        Player target = getServer().getPlayer(args[0]);
        if (target == null) {
            sender.sendMessage(ChatColor.RED + "Player not found.");
            return true;
        }

        // Unmute the player
        unmutePlayer(target);

        // Notify the sender
        sender.sendMessage(ChatColor.GREEN + "Unmuted player " + target.getName() + ".");

        return true;
    }

    return false;
}

@EventHandler
public void onPlayerChat(AsyncPlayerChatEvent event) {
    Player player = event.getPlayer();

    // Check if the player is muted
    if (mutedPlayers.containsKey(player)) {
        // Cancel the chat event
        event.setCancelled(true);

        // Display mute message
        player.sendMessage(ChatColor.RED + "You are currently muted for " + formatDuration(mutedPlayers.get(player)) + ".");
        player.sendMessage(ChatColor.RED + "Aaryan said shut up");
    }
}

private void mutePlayer(Player player, long duration) {
    // Add player to muted players map
    mutedPlayers.put(player, System.currentTimeMillis() + duration * 1000);

    // Log mute event
    getLogger().info(player.getName() + " was muted for " + duration + " seconds.");

    // Update player permissions
    player.sendMessage(ChatColor.RED + "You have been muted for " + formatDuration(duration) + ".");
    player.sendMessage(ChatColor.RED + "You will not be able to chat until your mute duration has expired.");
}

private void unmutePlayer(Player player) {
    // Remove player from muted players map
    mutedPlayers.remove(player);

    // Log unmute event
