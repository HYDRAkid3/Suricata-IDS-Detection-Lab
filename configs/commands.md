# Commands Reference

## Validate configuration
sudo suricata -T -c /etc/suricata/suricata.yaml

## Check rule files present (ET + local)
ls -lh /var/lib/suricata/rules

## View custom rules
cat /var/lib/suricata/rules/local.rules

## Evidence logs
sudo tail -n 20 /var/log/suricata/fast.log
sudo tail -n 5 /var/log/suricata/eve.json

## Packet validation (routing proof)
sudo tcpdump -i enp0s3
sudo tcpdump -i enp0s8
