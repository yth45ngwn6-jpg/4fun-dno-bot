require('dotenv').config();

const { Client, GatewayIntentBits, ChannelType } = require('discord.js');
const { joinVoiceChannel, VoiceConnectionStatus } = require('@discordjs/voice');

const TOKEN = process.env.DISCORD_TOKEN;
const CHANNEL_ID = '1570036937983791185';

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildVoiceStates
  ]
});

let connection;

async function joinRoom() {
  try {
    const channel = await client.channels.fetch(CHANNEL_ID);

    if (!channel || channel.type !== ChannelType.GuildVoice) {
      throw new Error('Voice channel not found');
    }

    if (!channel.permissionsFor(client.user).has(['ViewChannel', 'Connect'])) {
      throw new Error('Missing voice permissions');
    }

    connection = joinVoiceChannel({
      channelId: channel.id,
      guildId: channel.guild.id,
      adapterCreator: channel.guild.voiceAdapterCreator,
      selfMute: true,
      selfDeaf: true
    });

    connection.on('error', console.error);
    connection.on(VoiceConnectionStatus.Disconnected, () => {
      setTimeout(joinRoom, 5000);
    });

    console.log('Bot joined the voice channel!');
  } catch (error) {
    console.error(error);
  }
}

client.once('ready', joinRoom);

client.login(TOKEN);
