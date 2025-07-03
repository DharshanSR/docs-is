# Configure post-quantum TLS for inbound connections

By following this guide, you will enable **post-quantum TLS** in {{ product_name }} for inbound connections. An inbound connection refers to communication initiated by clients, such as browsers, applications, or APIs, connecting securely to {{ product_name }} using TLS.

!!! note "Post-quantum TLS requirements"

    - Post-quantum TLS is only supported over TLS 1.3.
    - {{product_name}} currently supports post-quantum security only on Linux and MacOS operating systems.

## Step 1: Build native libraries

For post-quantum TLS to work, {{product_name}} requires a few native libraries. These don't come pre-packaged with the {{product_name}} distribution as native libraries depend on the system architecture.

Use one of the following methods to build native libraries.

### Method 1: Use system libraries

This method relies on your operating system’s existing libraries during both the build process and runtime. This results in a faster build time and minimal disk space usage.

#### Build dependencies

{{product_name}} requires the following during the build process.

- Build tools (`make`, `cmake`, `wget`, `tar`, `autoconf`)
- GNU compiler (`gcc`)
- Apache Portable Runtime (APR) library
- OpenSSL 3.0 or higher.

To install these dependencies, follow the instructions below based on the operating system.

=== "Linux"

    1. Download the [source](https://www.openssl.org/source/) and follow the instructions given [here](https://github.com/openssl/openssl/blob/master/INSTALL.md#quick-installation-guide).

    2. Install the required build dependencies using the command for your relevant Linux distribution.

        In Debian-based Linux:

        ```bash
        apt-get install make cmake autoconf wget tar gcc libapr1-dev libssl-dev
        ```

        In Red Hat Linux distributions:

        ```bash
        yum install make cmake autoconf wget tar gcc apr-devel openssl-devel perl
        ```

=== "MacOS"

    1. Use Homebrew to install all the required build dependencies.

        ```bash
        brew install wget cmake autoconf openssl@3 apr
        ```

    2. Add the following line to your shell configuration file (e.g., `~/.bash_profile`, `~/.zshrc`, or `~/.bashrc`):

        For Intel-based Macs:

        ```bash
        export PATH="/usr/local/opt/apr/bin:$PATH"
        ```
        
        For Apple Silicon-based Macs:

        ```bash
        export PATH="/opt/homebrew/opt/apr/bin:$PATH"
        ```

#### Runtime dependencies

{{product_name}} requires the following during runtime.

1. The OpenSSL and APR libraries installed earlier.

2. Install the other dependencies by following the steps below:

    1. Shut down the {{product_name}} instance if running.
    2. Navigate to the `<IS_HOME>/bin/` folder and execute the following command:

        ```bash
        sh openssl-tls.sh --build_pqclib
        ```

        !!! note

            If you change the location of the {{product_name}} folder, you need to re-run the above command to reconfigure the runtime libraries.

### Method 2: Using self-contained libraries

This method installs all dependencies into the {{product_name}} directory, ensuring isolation from the system environment and highest compatibility across different systems and configurations.

#### Build dependencies

{{product_name}} requires the following during the build process.

- Build tools (`make`, `cmake`, `wget`, `tar`, `autoconf`)
- GNU compiler (`gcc`/`clang`)

To install these dependencies, follow the instructions below based on the operating system.

=== "Linux"

    Install the required build dependencies using the command for your relevant Linux distribution.

    In Debian-based Linux:

    ```bash
    apt-get install make cmake autoconf wget tar gcc git python3 autoconf libtool-bin
    ```

    In Red Hat Linux distributions:

    ```bash
    yum install make cmake autoconf wget tar gcc perl git python3 autoconf libtool
    ```

=== "MacOS"

    Use Homebrew to install dependencies.

    ```bash
    brew install wget autoconf cmake git python3 autoconf libtool
    ```

#### Runtime dependencies

{{product_name}} requires the following during runtime.

1. Shut down the {{product_name}} instance if running.
2. Navigate to the `<IS_HOME>/bin/` folder and execute the following command:

    ```bash
    sh openssl-tls.sh --build_openssl --build_pqclib
    ```

    !!! note
        If you change the location of the {{product_name}} folder, you need to re-run the above command to reconfigure the runtime libraries.

## Step 2: Enable post-quantum TLS

Follow the steps below to enable TLS to use post-quantum cryptography.

1. Shut down the {{product_name}} instance if running.
2. Add the following configuration to the `<IS_HOME>/repository/conf/deployment.toml` file.

    ``` toml
    [transport.https.openssl]
    enabled = true
    named_groups="X25519MLKEM768:x25519"

    [transport.https.sslHostConfig.properties]
    protocols="TLSv1+TLSv1.1+TLSv1.2+TLSv1.3"
    ```

3. Restart {{product_name}}.

!!! note "Disable post-quantum security"

    Make the following changes to the `<IS_HOME>/repository/conf/deployment.toml` file to disable post-quantum security.

    - Remove the configurations you added when enabling it and restart {{product_name}}.

    - You can keep using TLS 1.3 with post-quantum security disabled by adding the following configuration.
        
        ``` toml
        [transport.https.openssl]
        enabled = true

        [transport.https.sslHostConfig.properties]
        protocols="TLSv1+TLSv1.1+TLSv1.2+TLSv1.3"
        ```
