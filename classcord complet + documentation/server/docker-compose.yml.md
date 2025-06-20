services:
  classcord:
    build: .
    container_name: classcord-server
    ports:
      - "12345:12345"
    restart: unless-stopped
    networks:
      - classcord-net

networks:
  classcord-net:
    driver: bridge