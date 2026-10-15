# Chapter 6 Reflection

The Red Hat-based system required the most distribution-specific configuration because the project originally came from an AlmaLinux deployment and included firewalld, SELinux, Red Hat PHP package names, and Red Hat-specific PHP-FPM paths. Even though many of those settings already worked on AlmaLinux, they had to be separated from the common tasks so that the Debian system would not try to use them.

The PHP-FPM difference caused the most difficulty. Red Hat uses the `php-fpm` service and `/run/php-fpm/www.sock`, while Debian uses a versioned PHP-FPM service and socket. The playbook now detects the installed PHP version on Debian and builds the correct service name, configuration path, and socket path from that value.

To support a third Linux distribution, I would first identify its OS family, package names, service names, PHP-FPM paths, web-server account, database socket, firewall system, and security framework. I would then add another small set of distribution-specific variables and conditions while keeping the shared WordPress deployment logic unchanged. This would preserve the single-playbook design and avoid duplicating the entire project.
