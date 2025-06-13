### Nexus Repository Manager Deployment with Ansible

This Ansible project automates the installation and configuration of Sonatype Nexus Repository Manager 3 on Linux servers, with support for SSL/TLS certificate generation and Java keystore configuration.

---

#### **Key Features**
1. **Flexible Installation Modes**
   - Offline (uses pre-downloaded `.tar.gz` package)
   
2. **Automated SSL/TLS Setup**
   - Self-signed certificate generation
   - Support for custom certificates
   - Java keystore configuration

3. **Customizable Settings**
   - JVM memory allocation
   - Network ports
   - Security parameters
   - Organizational details for certificates

---

#### **Requirements**
1. **Control Machine**
   - Ansible 2.14+
   - Python 3.x
   - `community.crypto` and `community.general`
    ```bash
    ansible-galaxy collection install community.general community.crypto
    ```

2. **Target Servers**
   - Rocky Linux/RHEL (9 or 10)
   - Root access (configured in `ansible.cfg`)
   - Internet access (for online installation)
   - OpenJDK 17 (automatically installed)

---

#### **Quick Start**
1. **Prepare Inventory**
   ```ini
   # inv file
   [nexus]
   your-server-ip ansible_user=root
   ```

2. **Configure Variables**
   ```yaml
   # host_vars/your-server.yml
   instance_name: "nexus-prod"
   cert_dns_sans:
     - nexus.yourdomain.com
   cert_ip_sans:
     - 192.168.1.100
   nexus_jvm_xms_mbytes: 4096
   nexus_jvm_xmx_mbytes: 4096
   ```

3. **Run Deployment**
   ```bash
   ansible-playbook site.yml
   ```

---

#### **Customization Guide**
| Variable                  | Default Value          | Description                                                                 |
|---------------------------|------------------------|-----------------------------------------------------------------------------|
| `offline`                 | `false`                | Set to `true` for air-gapped environments                                  |
| `local_nexus_tar`         | -                      | Path to Nexus tarball (required for offline)                               |
| `nexus_base_dir`          | `/opt/sonatype`        | Installation directory                                                     |
| `application_port_ssl`    | `8443`                 | HTTPS port for Nexus                                                       |
| `nexus_jvm_xms_mbytes`    | `3072`                 | Initial JVM heap size (MB)                                                 |
| `nexus_jvm_xmx_mbytes`    | `3072`                 | Max JVM heap size (MB)                                                     |
| `setup_certs`             | `true`                 | Enable SSL certificate setup                                               |
| `self_signed`             | `true`                 | Generate self-signed certificates                                          |
| `keystore_password`       | `changeme`             | Java keystore password                                                     |

---

#### **Certificate Management**
**Self-Signed Setup (Automatic):**
1. Generates 4096-bit RSA private key
2. Creates CSR with SANs (DNS + IP)
3. Signs certificate with these details:
   - Country (US), Locality (Manhattan)
   - Organization: "ACME org"
   - SANs: `nexus.example.local`, `1.2.3.4`

**Custom Certificate Import:**
1. Set `self_signed: false`
2. Specify paths:
   ```yaml
   cert_import_path_privkey: "/path/to/your.key"
   cert_import_path_crt: "/path/to/cert.crt"
   ```

---

#### **Post-Installation**
1. **Access Nexus:**
   ```
   https://your-server:8443
   ```
2. **Default Credentials:**
   - Username: `admin`
   - Password: Check `nexus-base-dir/sonatype-work/nexus3/admin.password`

3. **Service Management:**
   ```bash
   systemctl status nexus    # Check service status
   journalctl -u nexus -f    # View real-time logs
   ```

---

#### **Troubleshooting**
- **Permission Issues:** Ensure `/opt/sonatype` owned by `nexus` user
- **SSL Errors:** Verify keystore path in `jetty-https.xml`
- **Memory Problems:** Adjust `nexus_jvm_xmx_mbytes` in host vars
- **Port Conflicts:** Check firewall rules for ports `8080`/`8443`

> **Note:** Always run `systemctl daemon-reload` after changing service files

---

#### **License**
This project uses Ansible roles from Ansible Galaxy. Commercial use of Nexus Repository Manager requires a license from Sonatype.