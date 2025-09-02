# 📝 Operating Systems with C – Cheat Sheet

This sheet contains **all main OS lab exercises** with inline explanations inside the code.  
Just study the patterns, and you’ll be ready for your exam.

---

## 1. Hello World

```c
#include <stdio.h>   // Standard I/O functions

int main() {
    printf("Hello, world!\n");  // Print message with newline
    return 0;                   // Return 0 means program success
}
//  to run it
clang -o hello hello.c
./hello  //


(a) Stdlib version – fopen, fread, fwrite
#include <stdio.h>
#include <stdlib.h>
#define BUFSIZE 1024*1024   // Buffer size: 1 MB

int main(int argc, char *argv[]) {
    FILE *in, *out;         // File pointers for source + destination
    char buf[BUFSIZE];      // Temporary storage buffer
    size_t rd, wr;          // Track bytes read/written

    // Expect exactly 2 arguments: source file and destination file
    if (argc != 3) {
        fprintf(stderr, "Usage: %s src dest\n", argv[0]);
        exit(1);
    }

    // Open source file in binary read mode
    in = fopen(argv[1], "rb");
    if (!in) { perror("open src"); exit(1); }

    // Open/create destination file in binary write mode
    out = fopen(argv[2], "wb");
    if (!out) { perror("open dest"); fclose(in); exit(1); }

    // Copy loop: read a chunk into buffer, then write it out
    while ((rd = fread(buf, 1, BUFSIZE, in)) > 0) {
        wr = fwrite(buf, 1, rd, out);
        if (wr != rd) { perror("short write"); break; }
    }

    fclose(out);  // Close destination
    fclose(in);   // Close source
    return 0;
}

(b) POSIX version – open, read, write
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>   // read(), write(), close()
#include <fcntl.h>    // open() flags
#define BUFSIZE 1024*1024

int main(int argc, char *argv[]) {
    int in, out;              // File descriptors (integers)
    char buf[BUFSIZE];        // Temporary buffer
    ssize_t rd, wr;           // Bytes read/written

    // Expect 2 arguments: source + destination
    if (argc != 3) {
        fprintf(stderr, "Usage: %s src dest\n", argv[0]);
        exit(1);
    }

    // Open source file (read-only)
    in = open(argv[1], O_RDONLY);
    if (in < 0) { perror("open src"); exit(1); }

    // Create destination file with rw-r--r-- permissions
    out = open(argv[2], O_CREAT|O_WRONLY|O_TRUNC, 0644);
    if (out < 0) { perror("open dest"); close(in); exit(1); }

    // Copy loop: read from source, write to destination
    while ((rd = read(in, buf, BUFSIZE)) > 0) {
        wr = write(out, buf, rd);
        if (wr != rd) { perror("write error"); break; }
    }

    close(in);   // Close source
    close(out);  // Close destination
    return 0;
}
