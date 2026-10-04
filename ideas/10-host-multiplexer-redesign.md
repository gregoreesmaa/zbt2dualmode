# Idea 10 — Ground-up host multiplexer redesign

## Proposal summary

Replace the CPC daemon (`cpcd`) sharing layer with a purpose-built,
version-pinned host multiplexer that carries the Zigbee stack
(`zigbeed`) and the Thread stack (`otbr-agent`) over one RCP on a
single ZBT-2. Keep the RCP-MultiPAN firmware concept but redesign the
host side from scratch: one multiplexer binary, one pinned
RCP/host-stack version set, explicit framing over the single UART
(EFR32MG24A420F1536IM40, RCP at 460800 baud per `zbt2_openthread_rcp.yaml`).

## Required design properties

- **Backpressure:** per-endpoint queues with drop/pause policy so a
  burst on one PAN cannot starve or overrun the other; UART driver
  must apply hardware flow control (RTS/CTS, cf. EUSART0 pinout in
  `03-firmware-inventory.md`).
- **Watchdog / recovery:** liveness heartbeats on each endpoint plus
  the UART; deterministic restart order (mux, then stacks) with
  bounded reconnect, not indefinite "secondary unresponsive" hangs.
- **Skew detection:** refuse to start on RCP/EZSP/Spinel version
  mismatch with a machine-readable error naming the expected trio;
  pin OpenThread commit host-side to the firmware commit.
- **Observability:** per-endpoint counters (frames, retries,
  drops, latency percentiles), structured logs with endpoint tags,
  and a status socket for health checks.

## Why the old cpcd failed these

Per `02-nabucasa-learnings.md`: `cpcd` added version coupling
(CPCd version conflicts), opaque socket/serial failure modes
("secondary unresponsive", `socket://core-silabs-multiprotocol:9999`
issues), and EZSP/CPC/RCP skew surfacing as flashing or
commissioning failures. It multiplexed bytes without visible
backpressure or endpoint-level metrics, so one stuck stack wedged
both, and debugging required correlating three moving parts.

## Effort / risk estimate

- **Effort:** high — new daemon plus protocol shims for both stacks,
  version-pinning harness, soak tests; roughly firmware-team scale,
  not a weekend patch.
- **Risk:** high — still single-radio timeslicing underneath, so the
  RF failure domain (two PANs, separate channels/keys on one front
  end) remains even if the host layer is fixed.

## Pros

- Removes the least-loved component while keeping one-stick hardware.
- Pinned versions and explicit errors convert silent wedges into
  actionable failures.
- Metrics make the two-PAN reliability claim testable with data.

## Cons

- Large build-and-maintain cost for a configuration the vendor
  measured and rejected.
- Does not fix RF timeslice loss; host robustness cannot recover
  airtime that does not exist.
- Forks from the Silicon Labs CPC ecosystem; tracks its changes manually.

## Verdict

Technically coherent but economically weak: do it only as a funded,
measured experiment with pass/fail two-PAN reliability gates, or not
at all. For production use today, prefer one radio per protocol
(see idea 01).
