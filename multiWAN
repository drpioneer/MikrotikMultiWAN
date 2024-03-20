# Script HA ISP Recursive Routing ISP Failover using DHCP by rextended
# APPLY TO '/ip dhcp-client add script=' section
# https://forum.mikrotik.com/viewtopic.php?p=1001987#p1001987
# tested on ROS 6.49.10 & 7.12
# updated 2024/03/20

:global HealthCheckIP 8.8.8.8; # IP to use to check if ISP path is working. Use different IPs for each ISP.
:global ISPPriority 1; # Which ISP path to use first. 1 is the highest priority. Each ISP needs a different priority value.
:global ISPName "ISP_$interface"; # This creates the comments for the routes and is used to find and change/delete the entries.

# Make sure to set Add Default Route to "no" on the DHCP Client.
/ip dhcp-client set [find where interface=$interface] add-default-route=no;
/ip route;

# Add Recursive Gateway Health Check IP Monitor
:local count [:len [find where comment="$ISPName_Monitor"]];
:if ($bound=1) do={
  :if ($count=0) do={
    add comment="$ISPName_Monitor" disabled=no distance=1 dst-address="$HealthCheckIP/32" gateway=$"gateway-address" scope=10 target-scope=10;
  } else={
    :if ($count=1) do={
      :local test [find where comment="$ISPName_Monitor"];
      :if ([get $test gateway]!=$"gateway-address") do={set $test gateway=$"gateway-address"}
    } else={:error "Multiple routes found"}}
} else={:remove [find where comment="$ISPName_Monitor"]}

# Add 0.0.0.0/0 route to ISP Gateway
:local count2 [:len [find where comment=$ISPName]];
:if ($bound=1) do={
  :if ($count2=0) do={
    add check-gateway=ping comment=$ISPName disabled=no distance=$ISPPriority dst-address=0.0.0.0/0 gateway=$HealthCheckIP scope=30 target-scope=11}
} else={:remove [find where comment=$ISPName]}
