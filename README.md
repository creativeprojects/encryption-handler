# encryption-handler

Proof of concept of a poor man's encryption over http (without using SSL/TLS)

## Server side

```go
package main

import (
	"crypto/rand"
	"fmt"
	"io"
	"log"
	"net/http"

	"github.com/creativeprojects/encryption"
)

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		writeString(w, "This message is going to be encrypted if your client supports it!")
	})

	port := 3001
	passphrase := []byte("I have been eating too much chocolate")

	// generate a random salt
	salt := make([]byte, 41)
	if _, err := io.ReadFull(rand.Reader, salt); err != nil {
		panic(err)
	}

	// create the encryption handler around your usual handler
	handler, err := encryption.NewHandler(passphrase, salt, mux)
	if err != nil {
		panic(err)
	}

	log.Printf("Service is listening on port %d...", port)
	log.Println(http.ListenAndServe(fmt.Sprintf(":%d", port), handler))
}

func writeString(w http.ResponseWriter, payload string) {
	w.Header().Set("Content-Type", "text/plain; charset=utf8")
	_, _ = io.WriteString(w, payload+"\n")
}


```

See the standard response:

```
% curl -v http://localhost:3001
```

See the encrypted response:

```
curl -v -H "Accept-Encoding: aesgcm" http://localhost:3001
```
