# Health signals (observed 26 Nov 2026, Open5GS 2.8.0, chart 2.4.1)

The agent's phase 4 checks are built from these real outputs.

## PFCP association (SMF ↔ UPF)
Source: `kubectl logs deployment/open5gs-smf -n open5gs`

| State | Log line |
|---|---|
| Server up | `[pfcp] INFO: pfcp_server() [<smf-pod-ip>]:8805` |
| Healthy | `[smf] INFO: PFCP associated [<upf-service-ip>]:8805` |
| Unhealthy | `[pfcp] WARNING: ... No Reponse. Give up! for step 1 type 5 peer [<upf>]:8805` |
| Unhealthy | `[smf] WARNING: Retry association with peer failed [<upf>]:8805` |

Note: "Reponse" is Open5GS's own spelling, so match it exactly.

## NRF registration
```
GET http://open5gs-nrf-sbi:7777/nnrf-nfm/v1/nf-instances?nf-type=SMF   (HTTP/2, prior knowledge)
```
- Healthy: `totalItemCount` >= 1, and the instance profile shows `"nfStatus":"REGISTERED"`
- Unhealthy: `totalItemCount: 0`
- The NF instance ID changes when the SMF re-registers, so check count and status, not a fixed ID.

## Pod health
Source: Kubernetes API (namespace `open5gs`)
- Healthy: Running, all containers ready
- Unhealthy: CrashLoopBackOff, stuck in Init, or restarts rising without a node reboot
- Restart counts also go up on every instance stop/start, so compare against the node's boot time.

## Addressing
- SMF PFCP server listens on its pod IP (10.42.0.0/16)
- SMF reaches the UPF through the UPF **Service IP** (10.43.0.0/16), which stays the same when the UPF pod is replaced