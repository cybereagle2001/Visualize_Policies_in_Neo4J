# Visualize your FW Logs using Neo4J

# Importing Logs
Everything starts by importing logs in CSV format and reformating it in a way we can 

## source based Database Modelisation
// Load CSV file skipping the header row
LOAD CSV WITH HEADERS FROM "file:///policies.csv" AS row

// Create Policy node once per policy
MERGE (p:Policy {name: row.Policy})
SET p += {
  action: row.Action,
  nat: row.NAT,
  type: row.Type,
  log: row.Log,
  bytes: row.Bytes
}

// Handle From zones
FOREACH (fromZone IN split(row.From, ",") |
  MERGE (fz:Zone {name: fromZone})
  MERGE (p)-[:FROM_ZONE]->(fz)
)

// Handle To zones
FOREACH (toZone IN split(row.To, ",") |
  MERGE (tz:Zone {name: toZone})
  MERGE (p)-[:TO_ZONE]->(tz)
)

// Handle Destinations
FOREACH (dest IN split(row.Destination, ",") |
  MERGE (d:Destination {name: dest})
  MERGE (p)-[:DESTINATION]->(d)
)

// Handle Schedule
FOREACH (sched IN split(row.Schedule, ",") |
  MERGE (s:Schedule {name: sched})
  MERGE (p)-[:SCHEDULE]->(s)
)

// Handle Services
FOREACH (svc IN split(row.Service, ",") |
  MERGE (srv:Service {name: svc})
  MERGE (p)-[:SERVICE]->(srv)
)

// Handle IP Pools
FOREACH (ignoreMe IN CASE WHEN size(split(row.`IP Pool`, ",")) = 1 AND split(row.`IP Pool`, ",")[0] = "" THEN [] ELSE split(row.`IP Pool`, ",") END |
  MERGE (ipn:IPPool {name: ignoreMe})
  MERGE (p)-[:USES_IP_POOL]->(ipn)
)

// Handle Security Profiles
FOREACH (prof IN split(row.`Security Profiles`, ",") |
  MERGE (sp:SecurityProfile {name: prof})
  MERGE (p)-[:SECURITY_PROFILE]->(sp)
)

// Now create Sources and link them directly to the Policy
FOREACH (src IN split(row.Source, ",") |
  MERGE (s:Source {name: src})
  MERGE (s)-[:APPLIES_TO]->(p)
);

## Source based Visualization
MATCH (s:Source)-[:APPLIES_TO]->(p:Policy)-[:DESTINATION]->(d:Destination)
return s,p,d
