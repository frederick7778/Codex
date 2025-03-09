# Codex
import requests
import time
import random
from bs4 import BeautifulSoup
from fake_useragent import UserAgent

# ===== CONFIGURATION =====
TARGET_URL = "https://singingfiles.com/show.php?l=0&u=2346217&id=63319"
NUM_CLICKS = 500
MIN_DELAY = 0.4  # Minimum delay in seconds
MAX_DELAY = 1.2  # Maximum delay in seconds
# ==========================

# List of common referers to rotate
REFERERS = [
    "https://www.google.com/",
    "https://www.facebook.com/",
    "https://twitter.com/",
    "https://www.youtube.com/",
    "https://www.reddit.com/"
]

# Fallback user-agents in case UserAgent fails
FALLBACK_USER_AGENTS = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Safari/605.1.15",
    "Mozilla/5.0 (X11; Linux x86_64; rv:89.0) Gecko/20100101 Firefox/89.0"
]

success_count = 0
failed_attempts = []

def get_random_headers():
    try:
        ua = UserAgent()
        user_agent = ua.random
    except:
        user_agent = random.choice(FALLBACK_USER_AGENTS)
    
    return {
        "User-Agent": user_agent,
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
        "Accept-Language": random.choice(["en-US,en;q=0.9", "fr-FR,fr;q=0.8", "de-DE,de;q=0.7"]),
        "Accept-Encoding": "gzip, deflate, br",
        "Referer": random.choice(REFERERS),
        "DNT": str(random.randint(0, 1)),
        "Connection": random.choice(["keep-alive", "close"])
    }

def simulate_clicks():
    global success_count
    for i in range(1, NUM_CLICKS + 1):
        try:
            headers = get_random_headers()
            
            # Add random cookies to appear more legitimate
            cookies = {
                "fake_cookie_{}".format(random.randint(1,5)): "value_{}".format(random.randint(1000,9999))
            }

            with requests.Session() as session:
                response = session.get(
                    TARGET_URL,
                    headers=headers,
                    cookies=cookies,
                    timeout=10,
                    allow_redirects=True
                )

                # Count as success if we get any 2xx status code
                if 200 <= response.status_code < 300:
                    success_count += 1
                    print(f"Attempt {i}/{NUM_CLICKS}: Success ✅ (Status: {response.status_code})")
                else:
                    raise Exception(f"HTTP {response.status_code}")

                # Random delay to appear more human
                delay = random.uniform(MIN_DELAY, MAX_DELAY)
                time.sleep(delay)

        except Exception as e:
            failed_attempts.append(i)
            print(f"Attempt {i}/{NUM_CLICKS}: Failed ❌ ({str(e)})")
            
            # Longer delay after failed attempts
            time.sleep(random.uniform(1.5, 3))

    # Generate final report
    print(f"\n=== Final Report ===")
    print(f"Total attempts: {NUM_CLICKS}")
    print(f"Successful clicks: {success_count} ({success_count/NUM_CLICKS*100:.2f}%)")
    print(f"Failed attempts: {len(failed_attempts)}")
    if failed_attempts:
        print(f"Failure positions: {failed_attempts[:10]}{'...' if len(failed_attempts) > 10 else ''}")

if __name__ == "__main__":
    print("=== Starting Click Simulation ===")
    simulate_clicks()
