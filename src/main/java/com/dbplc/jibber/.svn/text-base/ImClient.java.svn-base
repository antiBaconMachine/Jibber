/*
 * To change this template, choose Tools | Templates
 * and open the template in the editor.
 */
package com.dbplc.jibber;

import java.util.Arrays;
import java.util.List;
import org.jivesoftware.smack.Chat;

/**
 *
 * @author oedwards
 */
public interface ImClient {

    public static final String COMMAND_CHARACTER = "/";

    public enum Command {

        ROSTER {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                client.listFriends();
            }

												@Override
												public String getDescription() {
														return "List the user's roster";
												}
        },
        EXIT {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                client.exit();
            }

												@Override
												public String getDescription() {
														return "";
												}
        },
								//TODO allow for modes
        STATUS {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 2) {
                    String stat = tokens.get(1);
                    client.setStatus(stat, Command.concat(2, tokens));
                } else {
                    throw new IllegalArgumentException("Status should be in format \"/status newStatus [message]\"");
                }
            }

												@Override
												public String getDescription() {
														return "Set a new presence status";
												}
        },
        REQUEST {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 2) {
                    client.request(tokens.get(1));
                } else {
                    throw new IllegalArgumentException("no JID specified");
                }
            }

												@Override
												public String getDescription() {
														return "Request a JID as friend";
												}
        },
        ACCEPT {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 2) {
                    client.accept(tokens.get(1));
                } else {
                    throw new IllegalArgumentException("no username specified");
                }
            }

												@Override
												public String getDescription() {
														return "Accept a pending friend request";
												}
        },
        REJECT {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 2) {
                    client.reject(tokens.get(1));
                } else {
                    throw new IllegalArgumentException("no username specified");
                }
            }

												@Override
												public String getDescription() {
														return "Reject a pending friend request";
												}
        },
        REMOVE {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 2) {
                    client.remove(tokens.get(1));
                } else {
                    throw new IllegalArgumentException("no username specified");
                }
            }
												@Override
												public String getDescription() {
														return "Remove a friend";
												}
        },
        CHAT {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 2) {
                    client.startChat(tokens.get(1));
                } else {
                    client.listChats();
                }
            }

												@Override
												public String getDescription() {
														return "With no arguments lists all open chats. With a JID starts a new chat";
												}
        },
        TARGET {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 2) {
                    client.setTarget(tokens.get(1));
                } else {
                    client.displayNotification("Target is " + client.getTarget());
                }
            }

												@Override
												public String getDescription() {
														return "Set a chat as the current target";
												}
        },
        SEND {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 3) {
                    client.sendMessage(Command.concat(2, tokens), tokens.get(1));
                } else {
                    throw new IllegalArgumentException("send should be in form: /send username message");
                }
            }

												@Override
												public String getDescription() {
														return "Send a message to the specified user without changing the current target";
												}
        },
        SWITCH {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 3) {
                    client.sendMessage(Command.concat(2, tokens), tokens.get(1), true);
                } else {
                    throw new IllegalArgumentException("switch should be in form: /switch username message");
                }
            }

												@Override
												public String getDescription() {
														return "Send a message to the specified user and change the current target";
												}
        },
        ALIAS {

            @Override
            public void execute(ImClient client, List<String> tokens) {
                if (tokens.size() >= 3) {
                    client.alias(tokens.get(1), tokens.get(2));
                } else {
                    throw new IllegalArgumentException("alias should be in form: /alias username alias");
                }
            }

												@Override
												public String getDescription() {
														return "Alias a user";
												}
        },
        INFO {
            @Override
            public void execute(ImClient client, List<String> tokens) {
                client.info();
            }
												@Override
												public String getDescription() {
														return "Display pertinent client info including username, host, port etc";
												}
        },
				HELP {
						@Override
						public void execute(ImClient client, List<String> tokens) {
							for(Command c : Command.values()) {
									client.displayNotification(c.name() + " " +c.getDescription());
							}
						}

						@Override
								public String getDescription() {
										return "Display this command list";
								}
				},
				ROOMS {
					@Override
					public void execute(ImClient client, List<String> tokens) {
						String server = tokens.size() > 1 ? tokens.get(1) : null;
						List<Room> rooms = client.discoverRooms(server);
						for (Room r : rooms) {
							client.displayNotification(r.toString());
						}
					}

					@Override
					public String getDescription() {
						return "List available MUC rooms";
					}
				};

        public abstract void execute(ImClient client, List<String> tokens);

								public String getDescription() {
										return "";
								}

        public static List<String> tokenise(String input) {
            return Arrays.asList(input.substring(COMMAND_CHARACTER.length()).split(" "));
        }

        private static String concat(int startIndex, List<String> tokens) {
            StringBuilder msg = new StringBuilder();
            if (tokens.size() > startIndex) {
                for (String s : tokens.subList(startIndex, tokens.size())) {
                    msg.append(" ");
                    msg.append(s);
                }
                return msg.substring(1);
            }
            return "";
            
        }
    }

    public void listFriends();

    public void exit();

    public void setStatus(String stat, String msg);

    public void displayNotification(String msg);

    public void sendMessage(String message);

    public void sendMessage(String message, String username);

    public void sendMessage(String message, String username, boolean changeTarget);

    public void displayMessage(String from, String message);

    public void accept(String username);

    public void accept(String username, String alias);

    public void reject(String username);

    public void request(String username);

    public void request(String username, String alias);

    public void alias(String username, String alias);

    public void remove(String username);

    public Chat startChat(String username);

    public void setTarget(String username);

    public String getTarget();

    public void addChat(String username, Chat chat);

    public void removeChat(String username);

    public void listChats();

    public void info();

		public List<Room> discoverRooms(String server);

		public class Room {
		private String entityId;
		private String node;
		private String name;

		public Room(String entityId, String node, String name) {
			this.entityId=entityId;
			this.node=node;
			this.name=name;
		}

		@Override
		public String toString() {
			return name +" "+node+" "+entityId;
		}
	}
}
