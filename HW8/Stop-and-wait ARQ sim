import csv
import random
from pathlib import Path

PACKETS = 1000
RTT_MS = 100              # زمان یک تبادل موفق (ارسال + ACK)
TIMEOUT_MS = 200          # زمان انتظار تا تایمر منقضی شود (در صورت از دست رفتن)
ERROR_RATES = (0.05, 0.10)
MODEL_ACK_LOSS = False    # اگر True شود، ACK هم با همان نرخ خطا گم می‌شود
SEED = 20261114
MAX_ATTEMPTS_PER_PACKET = 10000


def simulate(error_rate: float):
    # seed وابسته به نرخ خطا است، نه ترتیب سناریوها
    rng = random.Random(f"{SEED}-{error_rate}")
    ack_error_rate = error_rate if MODEL_ACK_LOSS else 0.0
    records = []
    total_attempts = 0
    total_timeouts = 0
    elapsed_ms = 0

    for packet in range(1, PACKETS + 1):
        for attempt in range(1, MAX_ATTEMPTS_PER_PACKET + 1):
            total_attempts += 1
            data_lost = rng.random() < error_rate
            ack_lost = (not data_lost) and rng.random() < ack_error_rate
            if data_lost or ack_lost:
                total_timeouts += 1
                elapsed_ms += TIMEOUT_MS
            else:
                elapsed_ms += RTT_MS
                break
        else:
            raise RuntimeError(f"packet {packet} exceeded retry limit")

        records.append({
            "error_rate": error_rate,
            "packet": packet,
            "packet_attempts": attempt,
            "packet_retransmissions": attempt - 1,
            "result": "delivered",
        })

    # مقادیر نظری
    q = (1 - error_rate) * (1 - ack_error_rate)          # احتمال موفقیت هر تلاش
    exp_attempts = 1 / q
    exp_time_ms = RTT_MS + (exp_attempts - 1) * TIMEOUT_MS
    summary = {
        "error_rate": error_rate,
        "packets_delivered": PACKETS,
        "total_transmissions": total_attempts,
        "total_retransmissions": total_attempts - PACKETS,
        "timeouts": total_timeouts,
        "elapsed_ms": elapsed_ms,
        "mean_transmissions_per_packet": total_attempts / PACKETS,
        "throughput_packets_per_second": PACKETS / (elapsed_ms / 1000),
        "theory_mean_transmissions": exp_attempts,
        "theory_throughput_packets_per_second": 1000 / exp_time_ms,
    }
    return records, summary


def main():
    rows, summaries = [], []
    for rate in ERROR_RATES:
        scenario_rows, summary = simulate(rate)
        rows.extend(scenario_rows)
        summaries.append(summary)

    output = Path(__file__).with_name("stop_and_wait_results.csv")
    fields = ["record_type", "error_rate", "packet", "packet_attempts",
              "packet_retransmissions", "result", "packets_delivered",
              "total_transmissions", "total_retransmissions", "timeouts",
              "elapsed_ms", "mean_transmissions_per_packet",
              "throughput_packets_per_second", "theory_mean_transmissions",
              "theory_throughput_packets_per_second"]
    with output.open("w", newline="", encoding="utf-8") as handle:
        writer = csv.DictWriter(handle, fieldnames=fields)
        writer.writeheader()
        for row in rows:
            writer.writerow({"record_type": "packet", **row})
        for summary in summaries:
            writer.writerow({"record_type": "summary", **summary})

    print(f"Packets per scenario: {PACKETS}")
    print(f"RTT: {RTT_MS} ms | Timeout: {TIMEOUT_MS} ms | ACK loss modeled: {MODEL_ACK_LOSS}")
    print(f"Base seed: {SEED} (per-scenario seed derived from error rate)")
    for s in summaries:
        print(f"\nError rate: {s['error_rate']:.0%}")
        print(f"Packets delivered: {s['packets_delivered']}")
        print(f"Total transmissions: {s['total_transmissions']}")
        print(f"Retransmissions: {s['total_retransmissions']}")
        print(f"Timeouts: {s['timeouts']}")
        print(f"Elapsed time: {s['elapsed_ms']} ms")
        print(f"Mean transmissions/packet: {s['mean_transmissions_per_packet']:.3f} "
              f"(theory {s['theory_mean_transmissions']:.3f})")
        print(f"Throughput: {s['throughput_packets_per_second']:.3f} packets/s "
              f"(theory {s['theory_throughput_packets_per_second']:.3f})")
    print(f"\nCSV: {output}")


if __name__ == "__main__":
    main()
