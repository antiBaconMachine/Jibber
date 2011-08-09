package com.dbplc.jibber;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Collection;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.logging.Level;
import java.util.logging.Logger;
import org.jivesoftware.smack.Chat;
import org.jivesoftware.smack.ConnectionConfiguration;
import org.jivesoftware.smack.MessageListener;
import org.jivesoftware.smack.PacketListener;
import org.jivesoftware.smack.Roster;
import org.jivesoftware.smack.RosterEntry;
import org.jivesoftware.smack.RosterListener;
import org.jivesoftware.smack.XMPPConnection;
import org.jivesoftware.smack.XMPPException;
import org.jivesoftware.smack.filter.PacketFilter;
import org.jivesoftware.smack.packet.IQ;
import org.jivesoftware.smack.packet.Message;
import org.jivesoftware.smack.packet.Packet;
import org.jivesoftware.smack.packet.Presence;
import org.jivesoftware.smack.packet.Presence.Type;
import org.jivesoftware.smack.packet.RosterPacket;
import org.jivesoftware.smackx.ServiceDiscoveryManager;
import org.jivesoftware.smackx.packet.DiscoverItems;

/**
 * Hello world!
 *
 */
public class CommandLineClient implements ImClient {

    private XMPPConnection connection;
    private String target = null;
    private Map<String, Chat> chats = new HashMap<String, Chat>();
    private Roster roster;
		private String server;
		private Integer port;

    public CommandLineClient(String server, String username, String password) {
			this(server, 5222, username, password);
		}
    public CommandLineClient(String server, Integer port, String username, String password) {
        Logger.getLogger(CommandLineClient.class.getName()).setLevel(Level.WARNING);
				this.server = server;
				this.port = port;
        try {
												/*ConnectionConfiguration config = new ConnectionConfiguration(server, 5222);
												config.setSASLAuthenticationEnabled(false);
            connection = new XMPPConnection(config);*/
						connection = new XMPPConnection(new ConnectionConfiguration(server, port));
            connection.connect();
            connection.login(username, password, "dbprimary");

            roster = connection.getRoster();
            roster.setSubscriptionMode(Roster.SubscriptionMode.manual);
            roster.addRosterListener(new RosterListener() {

                @Override
                public void entriesAdded(Collection<String> clctn) {
                    Logger.getLogger(CommandLineClient.class.getName()).log(Level.INFO, "Entries added to roster: " + clctn);
                }

                @Override
                public void entriesUpdated(Collection<String> clctn) {
                    Logger.getLogger(CommandLineClient.class.getName()).log(Level.INFO, "Entries updated in roster: " + clctn);
                }

                @Override
                public void entriesDeleted(Collection<String> clctn) {
                    Logger.getLogger(CommandLineClient.class.getName()).log(Level.INFO, "Entries deleted from roster: " + clctn);
                }

                @Override
                public void presenceChanged(Presence prsnc) {
                    displayNotification("Prescence changed: " + prsnc.getFrom() + " " + prsnc);
                }
            });

            connection.addPacketListener(new PacketListener() {

                @Override
                public void processPacket(Packet packet) {
                    displayNotification("Friend request from " + packet.getFrom());
                }
            }, new PacketFilter() {

                @Override
                public boolean accept(Packet packet) {
                    if (packet instanceof Presence) {
                        if (((Presence) packet).getType().equals(Presence.Type.subscribe)) {
                            return true;
                        }
                    }
                    return false;
                }
            });

            connection.addPacketListener(new PacketListener() {

                @Override
                public void processPacket(Packet packet) {
                    displayNotification(packet.getFrom() + " accepted friend request");
                }
            }, new PacketFilter() {

                @Override
                public boolean accept(Packet packet) {
                    if (packet instanceof Presence) {
                        if (((Presence) packet).getType().equals(Presence.Type.subscribed)) {
                            return true;
                        }
                    }
                    return false;
                }
            });


        } catch (XMPPException ex) {
												System.out.println("Connection failed "+ex.getMessage());
            ex.printStackTrace();
												System.exit(1);
        }
    }

    @Override
    public void sendMessage(String message) {
        Logger.getLogger(CommandLineClient.class.getName()).log(Level.INFO, "sending message to " + target);
        if (target == null) {
            throw new RuntimeException("no target specified");
        }
        sendMessage(message, target);
    }

    @Override
    public void sendMessage(String message, String username) {
        try {
            Chat chat = chats.get(username);
            if (chat == null) {
                chat = startChat(username);
            }
            chat.sendMessage(message);
        } catch (XMPPException ex) {
            ex.printStackTrace();
        }
    }

    @Override
    public void sendMessage(String message, String username, boolean changeTarget) {
        if (changeTarget) {
            setTarget(username);
        }
        sendMessage(message, username);
    }

    public void setStatus(String status) {
        this.setStatus(status, null);
    }
				
    @Override
    public void setStatus(String status, String msg) {
        System.out.println("setting status to " + status + " with msg " + msg);
        Type t = Presence.Type.valueOf(status);

        Presence p = new Presence(t);
        if (msg != null) {
            p.setStatus(msg);
        }
        connection.sendPacket(p);
    }

    @Override
    public void listFriends() {
        Collection<RosterEntry> entries = roster.getEntries();
        for (RosterEntry entry : entries) {
            Presence p = roster.getPresence(entry.getUser());
            String status = "";
            String msg = p.getStatus() != null ? " (" + p.getStatus() + ")" : "";

            if (!p.isAvailable()) {
                status = "Unavailable";
            } else if (p.getMode() != null) {
                switch (p.getMode()) {
                    case xa:
                    case away:
                        status = "Away";
                        break;
                    case dnd:
                        status = "Busy";
                        break;
                }
            } else {
                status = "Available";
            }


            String name = entry.getName() != null ? entry.getName() + " (" + entry.getUser() + ")" : entry.getUser();
            System.out.println(name + " : " + status + msg);
        }
    }

    @Override
    public void exit() {
        connection.disconnect();
        System.exit(0);
    }

    @Override
    public void displayNotification(String msg) {
        System.out.println("*** " + msg + " ***");
    }

    @Override
    public void displayMessage(String from, String message) {
        RosterEntry r = roster.getEntry(from);
        if(r!=null && r.getName() != null){
            from = r.getName();
        }
        System.out.println("[" + from + "] : " + message);
    }

    @Override
    public void accept(String username) {
        Presence p = new Presence(Presence.Type.subscribed);
        p.setTo(username);
        connection.sendPacket(p);
        displayNotification("Accepted " + username);

        if (!roster.contains(username)
                || roster.getEntry(username).getStatus()
                == RosterPacket.ItemStatus.SUBSCRIPTION_PENDING) {
            request(username);
        }
    }

    @Override
    public void accept(String username, String alias) {
        throw new UnsupportedOperationException("Not supported yet.");
    }

    @Override
    public void reject(String username) {
        displayNotification("Rejected " + username);
    }

    @Override
    public void request(String username) {
        Presence p = new Presence(Presence.Type.subscribe);
        p.setTo(username);
        connection.sendPacket(p);
        displayNotification("Requested " + username);
    }

    @Override
    public void request(String username, String alias) {
        throw new UnsupportedOperationException("Not supported yet.");
    }

    @Override
    public void alias(String username, String alias) {
        roster.getEntry(username).setName(alias);
    }

    @Override
    public void remove(String username) {
        try {
            roster.removeEntry(roster.getEntry(username));
            Presence p = new Presence(Presence.Type.unsubscribe);
            p.setTo(username);
            connection.sendPacket(p);
            displayNotification(username + " removed");
        } catch (XMPPException ex) {
            Logger.getLogger(CommandLineClient.class.getName()).log(Level.SEVERE, null, ex);
        }
    }

    @Override
    public Chat startChat(String username) {
        if (target == null) {
            setTarget(username);
        }

        Chat chat = connection.getChatManager().createChat(username, new MessageListener() {

            public void processMessage(Chat chat, Message msg) {
                displayMessage(chat.getParticipant(), msg.getBody());
            }
        });

        addChat(username, chat);
        return chat;
    }

    @Override
    public void setTarget(String username) {
        target = username;
    }

    @Override
    public String getTarget() {
        return target;
    }

    @Override
    public void addChat(String username, Chat chat) {
        chats.put(username, chat);
    }

    @Override
    public void removeChat(String username) {
        chats.remove(username);
    }

    @Override
    public void listChats() {
        System.out.println("Open chats:");
        for (String c : chats.keySet()) {
            Chat chat = chats.get(c);
            System.out.println(chat.getParticipant());
        }
    }

     @Override
    public void info() {
        displayNotification("User : "+connection.getUser());
        displayNotification("Host : "+connection.getHost());
        displayNotification("Port : "+connection.getPort());
        displayNotification("Connection ID : "+connection.getConnectionID());
        displayNotification("Service name : "+connection.getServiceName());
    }

    public static void main(String[] args) {

        if (args.length < 3) {
            Logger.getLogger(CommandLineClient.class.getName()).log(Level.SEVERE, "args should be server username password");
            System.exit(1);
        }
				CommandLineClient client;
				if (args.length > 3) {
					client = new CommandLineClient(args[0], Integer.parseInt(args[1]), args[2], args[3]);
				} else {
        client = new CommandLineClient(args[0], args[1], args[2]);

				}
        BufferedReader stdin = new BufferedReader(new InputStreamReader(System.in));

        while (true) {
            try {
                String in = stdin.readLine();
                if (in.startsWith(ImClient.COMMAND_CHARACTER)) {
                    List<String> tokens = Command.tokenise(in);

                    if (tokens.isEmpty()) {
                        throw new IllegalArgumentException("no command specified");
                    }

                    Command command = ImClient.Command.valueOf(tokens.get(0).toUpperCase());

                    if (command == null) {
                        throw new IllegalArgumentException("unknown command");
                    }

                    command.execute(client, tokens);
                } else {
                    client.sendMessage(in);
                }
            } catch (IllegalArgumentException ex) {
                Logger.getLogger(CommandLineClient.class.getName()).log(Level.SEVERE, null, ex);
            } catch (IOException ex) {
                Logger.getLogger(CommandLineClient.class.getName()).log(Level.SEVERE, null, ex);
            } catch (Exception ex) {
                Logger.getLogger(CommandLineClient.class.getName()).log(Level.SEVERE, null, ex);
            }
        }
    }

	@Override
	public List<Room> discoverRooms(String server) {
			List<Room> rooms = new ArrayList<Room>();
			if (server == null) {
				server = "muc." + this.server;
			}
			try{
				// Obtain the ServiceDiscoveryManager associated with my XMPPConnection
				ServiceDiscoveryManager discoManager = ServiceDiscoveryManager.getInstanceFor(connection);

				// Get the items of a given XMPP entity
				// This example gets the items associated with online catalog service
				DiscoverItems discoItems = discoManager.discoverItems(server);

				// Get the discovered items of the queried XMPP entity
				Iterator it = discoItems.getItems();
				// Display the items of the remote XMPP entity
				while (it.hasNext()) {
					DiscoverItems.Item item = (DiscoverItems.Item) it.next();
					rooms.add(new Room(item.getEntityID(), item.getNode(), item.getName()));
				}
		} catch (XMPPException ex) {
			ex.printStackTrace();
		}
		return rooms;
	}
}
