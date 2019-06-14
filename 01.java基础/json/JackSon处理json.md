### JackSon处理json ###

ObjectMapper mapper = new ObjectMapper();  
Email email = mapper.readValue(message, Email.class);