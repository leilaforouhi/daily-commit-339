import re
import argparse
from collections import Counter
from pathlib import Path


STATUS_PATTERN = re.compile(r'"\s*(\d{3})\s+')


def analyze_log_file(file_path):
    path = Path(file_path)

    if not path.is_file():
        raise FileNotFoundError(f"Log file not found: {path}")

    status_codes = Counter()
    total_lines = 0
    unmatched_lines = 0

    with path.open("r", encoding="utf-8", errors="replace") as file:
        for line in file:
            total_lines += 1
            match = STATUS_PATTERN.search(line)

            if match:
                status_codes[match.group(1)] += 1
            else:
                unmatched_lines += 1

    return {
        "total_lines": total_lines,
        "parsed_requests": sum(status_codes.values()),
        "unmatched_lines": unmatched_lines,
        "status_codes": dict(sorted(status_codes.items())),
    }


def display_report(report):
    print("\nServer Log Analysis")
    print("-" * 30)
    print(f"Total lines: {report['total_lines']}")
    print(f"Parsed requests: {report['parsed_requests']}")
    print(f"Unmatched lines: {report['unmatched_lines']}")

    print("\nHTTP Status Codes:")

    if not report["status_codes"]:
        print("No status codes detected.")
        return

    for code, count in report["status_codes"].items():
        print(f"HTTP {code}: {count}")

    errors = sum(
        count
        for code, count in report["status_codes"].items()
        if code.startswith(("4", "5"))
    )

    print(f"\nClient/server errors: {errors}")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Analyze HTTP status codes in a server log."
    )
    parser.add_argument("file", help="Path to the server log file")
    args = parser.parse_args()

    try:
        report = analyze_log_file(args.file)
        display_report(report)
    except OSError as error:
        print(f"Error: {error}")
