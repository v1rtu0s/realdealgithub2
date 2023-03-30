<script>
  export function generateBytes({
    ServerSeed,
    ClientSeed,
    nonce,
    cursor,
    count
  }) {
    // Setup cursor variables
    let currentRound = Math.floor(cursor / 32);
    let currentRoundCursor = cursor;
    currentRoundCursor -= currentRound * 32;
    // Generate bytes until count is fulfilled
    const bytes = [];
    while (bytes.length < count) {
      // HMAC function used to output provided inputs into bytes
      const hmac = crypto.createHmac("sha256", ServerSeed);
      hmac.update(`${ClientSeed}:${nonce}:${currentRound}`);
      const buffer = hmac.digest();
      // Update cursor for next iteration of loop
      while (currentRoundCursor < 32 && bytes.length < count) {
        bytes.push(buffer[currentRoundCursor]);
        currentRoundCursor += 1;
      }
      currentRoundCursor = 0;
      currentRound += 1;
    }
    console.log("bytes:", bytes);
    return bytes;
  }
</script>