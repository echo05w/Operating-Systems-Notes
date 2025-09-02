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


## 3. File Length (`fseek` + `ftell`)

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    FILE *f; 
    long len;

    // Need exactly 1 argument: file name
    if (argc != 2) {
        fprintf(stderr, "Usage: %s file\n", argv[0]);
        exit(1);
    }

    f = fopen(argv[1], "rb");   // Open file for reading
    if (!f) { perror("open"); exit(1); }

    // Move position to the end of file
    fseek(f, 0, SEEK_END);

    // Current position = file size in bytes
    len = ftell(f);
    printf("Length: %ld bytes\n", len);

    fclose(f);  // Close file
    return 0;
}


4. Directory Listing (opendir + readdir)
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>     // For opendir(), readdir(), closedir()
#include <sys/stat.h>   // For stat() to get file size

int main() {
    DIR *d; 
    struct dirent *entry;  // Directory entry
    struct stat st;        // File info (size, etc.)

    // Open current directory
    d = opendir(".");
    if (!d) { perror("opendir"); exit(1); }

    // Read entries one by one
    while ((entry = readdir(d)) != NULL) {
        // Only handle regular files (ignore dirs, links, etc.)
        if (entry->d_type == DT_REG) {
            // Get file information (size in bytes)
            if (stat(entry->d_name, &st) == 0) {
                printf("%s %ld\n", entry->d_name, st.st_size);
            }
        }
    }

    closedir(d);  // Close directory
    return 0;
}


## 5. Disk Usage (`du`-like program)

This program starts at a given directory (argument) and **recursively sums the sizes of all regular files** inside it.

```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>     // For opendir(), readdir()
#include <sys/stat.h>   // For stat()
#include <string.h>     // For string operations
#include <unistd.h>     // For chdir()

// Recursive function to compute total size
long sum_dir(const char *path) {
    DIR *d;
    struct dirent *entry;
    struct stat st;
    long total = 0;
    char buf[4096];

    d = opendir(path);
    if (!d) {
        perror(path);  // Print error if directory can’t be opened
        return 0;
    }

    while ((entry = readdir(d)) != NULL) {
        // Skip . and ..
        if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0)
            continue;

        // Build full path: path + "/" + entry->d_name
        snprintf(buf, sizeof(buf), "%s/%s", path, entry->d_name);

        if (stat(buf, &st) == 0) {
            if (S_ISREG(st.st_mode)) {
                // Regular file → add size
                total += st.st_size;
            } else if (S_ISDIR(st.st_mode)) {
                // Directory → recurse into it
                total += sum_dir(buf);
            }
            // Special files (devices, sockets, etc.) are ignored
        } else {
            perror("stat");
        }
    }

    closedir(d);
    return total;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s directory\n", argv[0]);
        exit(1);
    }

    long total = sum_dir(argv[1]);
    printf("Total size: %ld bytes\n", total);

    return 0;
}


## 6. Program Analysis (`xchg.c`)

This exercise is about analyzing and improving a given program (`xchg.c`) that tries to **swap the contents of two files**.

---

### (a) Purpose & parameters
- **Purpose:** exchange the contents of two files.  
- **Parameters:** the program requires two filenames as arguments.  

Example usage:
```bash
./xchg file1.txt file2.txt

(b) Tests & flaws

Case 1: Files same size → likely works.

Case 2: First larger → second file may be truncated incorrectly.

Case 3: First smaller → second file may keep extra data.

Other flaws:

Error handling may be missing.

Inefficient for large files (might copy byte by byte).

Doesn’t always truncate the target properly.

(c) Improved version → xchg2.c
#include <stdio.h>
#include <stdlib.h>

#define BUFSIZE 1024*1024   // Use a large buffer for efficiency

// Helper: copy contents from src → dst
void copy(FILE *src, FILE *dst) {
    char buf[BUFSIZE];
    size_t rd;
    while ((rd = fread(buf, 1, BUFSIZE, src)) > 0) {
        fwrite(buf, 1, rd, dst);
    }
}

int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(stderr, "Usage: %s file1 file2\n", argv[0]);
        exit(1);
    }

    // Open both files for reading
    FILE *f1 = fopen(argv[1], "rb");
    FILE *f2 = fopen(argv[2], "rb");
    if (!f1 || !f2) { perror("open"); exit(1); }

    // Create a temporary file in RAM or /tmp
    FILE *tmp = tmpfile();
    if (!tmp) { perror("tmpfile"); exit(1); }

    // Step 1: Copy file1 → tmp
    copy(f1, tmp);
    rewind(f1);
    rewind(f2);
    rewind(tmp);

    // Step 2: Truncate file1 and file2 for rewriting
    f1 = freopen(argv[1], "wb+", f1);
    f2 = freopen(argv[2], "wb+", f2);

    // Step 3: Copy file2 → file1
    copy(f2, f1);
    rewind(f2);
    rewind(tmp);

    // Step 4: Copy tmp (old file1) → file2
    copy(tmp, f2);

    // Close all files
    fclose(f1);
    fclose(f2);
    fclose(tmp);

    return 0;
}
