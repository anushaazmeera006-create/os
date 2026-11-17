--- /home/anushaazmeera/assignment/Q4/.q4_before/kernel_sysproc.c	2026-10-19 18:34:32.413987891 +0000
+++ kernel/sysproc.c	2026-10-19 18:40:30.283299394 +0000
@@ -36,31 +36,27 @@
   return kwait(p);
 }
 
+// Q4: Grow the heap lazily and shrink it by freeing mapped pages.
 uint64
 sys_sbrk(void)
 {
   uint64 addr;
-  int t;
   int n;
 
   argint(0, &n);
-  argint(1, &t);
   addr = myproc()->sz;
 
-  if (t == SBRK_EAGER || n < 0) {
-    if (growproc(n) < 0) {
+  if (n < 0) {
+    // Q4: Free mapped pages when the process shrinks its heap.
+    if (growproc(n) < 0)
       return -1;
-    }
   } else {
-    // Lazily allocate memory for this process: increase its memory
-    // size but don't allocate memory. If the processes uses the
-    // memory, vmfault() will allocate it.
-    if (addr + n < addr)
-      return -1;
-    if (addr + n > TRAPFRAME)
+    // Q4: Increase the heap size without allocating physical pages.
+    if (addr + n < addr || addr + n > TRAPFRAME)
       return -1;
     myproc()->sz += n;
   }
+
   return addr;
 }
 
--- /home/anushaazmeera/assignment/Q4/.q4_before/Makefile	2026-10-19 18:34:32.458508202 +0000
+++ Makefile	2026-10-19 18:38:10.297896202 +0000
@@ -149,6 +149,7 @@
 	$U/_vatest\
 	$U/_memtest\
 	$U/_vmprinttest\
+	$U/_lazytest\
 
 fs.img: mkfs/mkfs README $(UPROGS)
 	mkfs/mkfs fs.img README $(UPROGS)
--- /dev/null	2026-10-19 17:02:10.261713691 +0000
+++ user/lazytest.c	2026-10-19 18:42:28.472184780 +0000
@@ -0,0 +1,88 @@
+#include "kernel/types.h"
+#include "kernel/stat.h"
+#include "user/user.h"
+
+// Q4: Prints physical and process memory statistics.
+static void
+printmem(char *label)
+{
+  struct meminfo info;
+
+  if (getmeminfo(&info) < 0) {
+    printf("lazytest: getmeminfo failed\n");
+    exit(1);
+  }
+
+  printf("%s: free=%d user_pages=%d page_tables=%d\n",
+         label, (int)info.free_pages, (int)info.user_pages,
+         (int)info.page_table_pages);
+}
+
+// Q4: Tests lazy heap allocation, kernel buffer faults, fork, and invalid access.
+int
+main(void)
+{
+  int region_size = 32 * 1024 * 1024;
+  char *region;
+
+  printmem("before lazy sbrk");
+
+  // Q4: Reserve 32 MB without touching or allocating its pages.
+  region = sbrk(region_size);
+  if (region == (char *)-1) {
+    printf("lazytest: sbrklazy failed\n");
+    exit(1);
+  }
+  printmem("after lazy sbrk");
+
+  // Q4: Touch three widely spaced pages in the reserved region.
+  region[0] = 'A';
+  region[4096 * 4096] = 'B';
+  region[4096 * 8191] = 'C';
+  printmem("after touching three pages");
+
+  // Q4: Write from an untouched page to test lazy allocation in copyin.
+  printf("testing write with an untouched lazy page\n");
+  if (write(1, &region[4096 * 1024], 1) != 1) {
+    printf("lazytest: write from lazy page failed\n");
+    exit(1);
+  }
+  printmem("after write buffer fault");
+
+  int pid = fork();
+  if (pid < 0) {
+    printf("lazytest: fork failed\n");
+    exit(1);
+  }
+
+  if (pid == 0) {
+    // Q4: Fault in a child page that was still lazy after fork.
+    region[4096 * 3072] = 'D';
+    pause(20);
+    exit(0);
+  }
+
+  printmem("parent while child exists");
+  wait(0);
+  printmem("parent after child reaped");
+
+  pid = fork();
+  if (pid < 0) {
+    printf("lazytest: invalid-access fork failed\n");
+    exit(1);
+  }
+  if (pid == 0) {
+    // Q4: Fault beyond the process size so only this child is killed.
+    *(volatile char *)0x7fffffff = 1;
+    exit(0);
+  }
+  wait(0);
+  printf("invalid-access child was reaped\n");
+
+  // Q4: Shrink the reserved heap and report the final memory count.
+  sbrk(-region_size);
+  printmem("after shrinking heap");
+
+  exit(0);
+}
+
