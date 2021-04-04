# Docker security cheeat sheet
https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Docker_Security_Cheat_Sheet.md

To detect containers with known vulnerabilities - scan images using static analysis tools.

Free
    Clair
    Trivy

Commercial
    Snyk (open source and free option available)
    anchore (open source and free option available)
    Aqua Security's MicroScanner (free option available for rate-limited number of scans)
    JFrog XRay
    Qualys
    
To detect misconfigurations in Kubernetes:

    kubeaudit
    kubesec.io
    kube-bench

To detect misconfigurations in Docker:

    inspec.io
    dev-sec.io