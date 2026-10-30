# Submission - Smart Lead Capstone

> Copy this file into the root of your solution repository and fill it in. Keep it
> honest and concise - the reviewer reads this first.

- Name: skool.com/@john-rugge-4911
- **Solution repository:**  repo: https://github.com/jrrugge/devops-exercises1
-                           Smart Lead Capstone: https://github.com/jrrugge/devops-exercises1/tree/main/06-docker/09-capstone-smart-lead
- **Date:** : 30/10/2026

## How to run it

The reviewer will run exactly what you put here, on a machine with only Docker.

```bash
# clone
git clone https://github.com/jrrugge/devops-exercises1.git
cd devops-exercises1/06-docker/09-capstone-smart-lead

rename the provided env.example file to env, the config override - in cloud deployments secrets would be set in secrets manager
cp env.example .env


# bring the whole system up
docker compose up --build -d
<img width="1025" height="155" alt="image" src="https://github.com/user-attachments/assets/89be90bb-1f22-49af-9c29-37aeac6cf1c5" />


```
- **As above first set first env.example` to `.env`):**
  
- **URL / port the API is on:**
- App receives messages from potential leads on: http://localhost:8080/api/v1/messages
- Qualified leads can be fetched here: http://localhost:8080/api/v1/leads

- # Submit a message to populate the queue and database
curl -X POST http://localhost:8080/api/v1/messages \
  -H "Content-Type: application/json" \
  -d "{\"content\":\"How much does the pro plan cost?\"}"
See example below
<img width="1076" height="210" alt="image" src="https://github.com/user-attachments/assets/bd338679-5c00-487f-8569-5ab416803b17" />




# Verify the qualified lead was safely generated and stored
curl http://localhost:8080/api/v1/leads
See example below



## Prove it works

Paste the commands you use to demonstrate the end-to-end flow, and what you expect to
see:
<img width="1076" height="210" alt="image" src="https://github.com/user-attachments/assets/bd338679-5c00-487f-8569-5ab416803b17" />
```bash
# submit a message
curl -X POST localhost:8080/api/v1/messages \
  -H 'Content-Type: application/json' \
  -d '{"content":"How much does the pro plan cost?"}'

# then a lead should appear
curl localhost:8080/api/v1/leads
<img width="1175" height="197" alt="image" src="https://github.com/user-attachments/assets/f9047fd5-e546-42d4-ba7a-e5e016dd882d" />

```

## Design decisions (the important part)

Explain the choices you made and why. A few sentences each.

1. **Building the app image** - what base image(s), single or multi-stage, and why:
The pom.xml file dictates our base image choices because it lists the strict technical requirements the app needs to compile and run.
Inside the pom.xml, the <java.version>21</java.version> tag explicitly tells us the app is coded in Java 21. Because of this rule, we are forced to use Docker images that natively support Java 21, leading directly to our choices of maven:3.9.6-eclipse-temurin-21-jammy for compiling the code and eclipse-temurin:21-jre-jammy for running it.
We separated the compilation from the execution i.e multi-stage build. By discarding the heavy Maven tools and source code after building the jar file, and copying only the final app jar into a lightweight JRE runner, we keep the final image incredibly lean, small, and secure for production.
By reading these rules directly from the project's blueprint i.e pom file, we ensure our containers match the exact environment the application expects.

3. **Pointing the app at the containers** - which `application.properties` values you
   overrode, and how you passed them in:
I used spring boot's 'relaxed binding' mechanism to override the defaults inside application.properties by injecting uppercase environment variables directly through docker-compose.yml file, the docker-compose file also uses dynamic variable which point to the .env file to keep secret from being pushed into code repos.
The appplication.properties file shows the default network settings point to localhost for all services so I redirected network traffic away from localhost by pointing SPRING_DATASOURCE_URL and SPRING_CLOUD_AWS_SQS_ENDPOINT directly to our internal container service hostnames (db and localstack).
I then set ENABLE_FAKE_HF=true to toggle the offline AI stub, mirrored a literal upstream typo via AWS_QUEQUE_URL, and muted background connection noise by setting the MANAGEMENT_OTLP export variables to false for ease reviewing logs directly related to the application during build and troubleshooting.

4. **Startup ordering** - how you made the app wait for the database and queue to be
   *ready* (not just started):
I did this by using docker compose health checks per service paired with dependency routing.
Instead of a basic depends_on which only tracks if a process has started, I added explicit validation scripts using pg_isready for the db and a core curl -f request against LocalStack health diagnostic endpoint (/_localstack/health).
By setting the app’s depends_on config to require condition: service_healthy for both backing components, the Spring Boot container automatically remains in a locked wait-state, preventing startup crashes until the infrastructure sockets are fully initialized and ready to accept traffic.

5. **The queue** - how it gets created and how the app finds it:
The SQS queue is created automatically on startup by leveraging LocalStack's native container engine configuration via the SQS_QUEUES=message-analysis-queue environment variable in our Compose file.
The application then dynamically discovers this queue over the internal Docker network mesh by using Spring's Relaxed Binding properties (AWS_QUEQUE_URL and APP_QUEUE_NAME), which resolve straight to the LocalStack service container endpoint.

7. **The AI call** - how you ran without a real HuggingFace key, and how a real key
   would be supplied:
I ran the application without a real HuggingFace key by setting the ENABLE_FAKE_HF=true environment variable in our docker-compose.yml file, which instantly toggles the project’s built-in offline mock analyzer.
To use a real key in production without editing the image or committing secrets, a developer would switch that variable to 'false' and securely pass the token via an untracked local .env file mapped to HF_TOKEN, auto picked up by spring boot at boot time.

8. **Persistence** - where the database data lives and how you confirmed it survives a
   restart:

The database data physically lives inside the container at /var/lib/postgresql/data, which is mapped out to an isolated, docker-managed 'named disk volume called postgres_data on the host 'my local machine'.
[postgres_data:/var/lib/postgresql/data]
Confirmed durability by writing a message using curl, verifying it created a lead, and then executing a strict docker compose down to completely delete the running container infrastructure from memory.
After running docker compose up -d to rebuild the stack, a final curl http://localhost:8080/api/v1/leads request returned the identical records, proving the data survives independent container reboots. See below

<img width="1177" height="480" alt="image" src="https://github.com/user-attachments/assets/ba186908-26c9-4589-a7a3-0ef759ac51f2" />

## Reflection

Answer the questions from the capstone README:

1. Which `application.properties` values did you override, and why those exactly?
SPRING_DATASOURCE_URL & SPRING_CLOUD_AWS_SQS_ENDPOINT: Redirected away from localhost to point directly to our container service hostnames (db and localstack) so the containers could communicate across the docker internal bridge network mesh, enabling the app to run anywhere docker is installed.
AWS_QUEQUE_URL: Overridden exactly in the original file to safely route spring SQS listeners.
ENABLE_FAKE_HF: Switched to true to activate the offline mock analyzer stub, ensuring the platform works immediately for peer review purposes without external API keys.
3. How did you guarantee the app does not start before the database and queue are ready,
   and how is that different from a plain `depends_on`? see 4. health checks and dependency routing
4. Where does the database data live, and how did you prove it survives a restart? see 8. persistence above
5. How would you switch from the AI stub to a real HuggingFace key without changing the
   image or committing a secret? see 7 above.
6. What did you do to keep the final image small and safe, and what would you improve
   with more time?
With more time i would set up an Nginx reverse proxy container layer to safely handle upstream SSL/TLS certificate termination.
## Known limitations / what I would do next

-
