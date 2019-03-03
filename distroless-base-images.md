# What are distroless base images?

Someone could suggest using Alpine, Well that a valid and recommended suggestion! But Alpline is basically a Linux kernel (with an unofficial port of grsecurity patch), the musl C library, BusyBox, LibreSSL, and OpenRC! Alpine also uses its own package manager called apk-tools which can be used to install your needed runtime, the JRE in our case! But here's the question: Will you ever need that package manager again? Probably not, and then it just stays there, being a huge security risk.

This is where Google’s distroless images come in. "Distroless" images contain only your application and its runtime dependencies. They don’t contain any programs like shells and package managers usually found in a Linux distribution.
