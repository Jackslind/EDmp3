import hashlib
import time
from datetime import datetime, timedelta

def generate_exclusive_download_link(file_id: str, secret_key: str, expires_in_minutes: int = 60) -> str:
    """
    Generates a mock exclusive download link with a simple time-based signature.
    In a real system, this would involve a robust signing mechanism and token management.

    Args:
        file_id (str): A unique identifier for the MP3 file.
        secret_key (str): A secret key known only to the server for signing.
        expires_in_minutes (int): How long the link should be valid, in minutes.

    Returns:
        str: A mock download URL.
    """
    expiration_time = datetime.now() + timedelta(minutes=expires_in_minutes)
    timestamp = int(expiration_time.timestamp())

    # Create a simple "signature" - in a real system, this would be much more robust
    # e.g., HMAC with more parameters (user ID, IP, etc.)
    signature_string = f"{file_id}-{timestamp}-{secret_key}"
    signature = hashlib.sha256(signature_string.encode('utf-8')).hexdigest()

    # Construct a mock URL. In a real system, this would point to your actual download server endpoint.
    base_url = "https://edmp3.com/download"
    download_url = f"{base_url}?id={file_id}&timestamp={timestamp}&sig={signature}"

    return download_url

# --- Example Usage ---
if __name__ == "__main__":
    my_secret_server_key = "super_secure_secret_edmp3_key_123" # NEVER hardcode in real app! Use environment variables.
    mp3_file_id = "exclusive_track_001_artist_A"

    # Generate a link that expires in 5 minutes
    link = generate_exclusive_download_link(mp3_file_id, my_secret_server_key, expires_in_minutes=5)
    print(f"Generated Exclusive Download Link:\n{link}\n")
    print(f"This link is theoretically valid for 5 minutes from now ({datetime.now().strftime('%H:%M:%S')}).\n")

    # --- Server-side verification (conceptual) ---
    # When a user clicks the link, your server would perform this check:
    def verify_download_link(url: str, secret_key: str) -> bool:
        from urllib.parse import urlparse, parse_qs
        parsed_url = urlparse(url)
        query_params = parse_qs(parsed_url.query)

        received_file_id = query_params.get('id', [''])[0]
        received_timestamp_str = query_params.get('timestamp', [''])[0]
        received_signature = query_params.get('sig', [''])[0]

        if not all([received_file_id, received_timestamp_str, received_signature]):
            print("Missing parameters in URL.")
            return False

        try:
            received_timestamp = int(received_timestamp_str)
        except ValueError:
            print("Invalid timestamp.")
            return False

        # Check expiration
        if datetime.now().timestamp() > received_timestamp:
            print("Link has expired.")
            return False

        # Re-generate expected signature and compare
        expected_signature_string = f"{received_file_id}-{received_timestamp}-{secret_key}"
        expected_signature = hashlib.sha256(expected_signature_string.encode('utf-8')).hexdigest()

        if expected_signature == received_signature:
            print("Signature valid. File can be downloaded.")
            return True
        else:
            print("Signature invalid. Access denied.")
            return False

    print("--- Server-side verification simulation ---")
    is_valid = verify_download_link(link, my_secret_server_key)
    print(f"Link is valid: {is_valid}")

    # Simulate an expired link (for demonstration, just change the timestamp)
    print("\n--- Simulating an expired link ---")
    # Manually create a link with an old timestamp
    expired_timestamp = int((datetime.now() - timedelta(minutes=10)).timestamp())
    expired_signature_string = f"{mp3_file_id}-{expired_timestamp}-{my_secret_server_key}"
    expired_signature = hashlib.sha256(expired_signature_string.encode('utf-8')).hexdigest()
    expired_link = f"https://edmp3.com/download?id={mp3_file_id}&timestamp={expired_timestamp}&sig={expired_signature}"
    print(f"Simulated Expired Link:\n{expired_link}")
    is_valid_expired = verify_download_link(expired_link, my_secret_server_key)
    print(f"Link is valid (after simulated expiration): {is_valid_expired}")
