===============================================================================================
                                    Guardrails & Policy
===============================================================================================

Compliance

Remediation

Events

Initiative -- Collection of Policies


=============
=============

- Authoring
    Assignments
    Definations
        US Locations Policy
        Allowed Storage Account SKUs Policy
    Exemptions

=============
Namming & Tagging Standards
=============

=============
Templates and parameter files
=============

Selection Category of Template

Terraform --> Provider Plugin --> Provisioning

Vairables.tf --> Parameters

main.tf --> Templates

Output.tf --> Result of provisioned

In the hybrid case terraform is the best option, but In-case of Single Cloud i.e. Azure, We can go with the native cloud provisioning tool.

========================
VM and OS Configurations
========================

Configuration

Powershell DSC
Chef, Puppet, and Ansible

Declarative
Imperative

=========================
Custom Images, and Packer
=========================

Customized Images can be deployed with Packer, and also can we versioned

Base Image & It should be customized.

========================
Container & Docker
========================

Docker Base Image --> Layers of on top of base image --> Upon building will get the docker image.

========================
Kubernetes - Large Orchestration
========================

Container --> Pods ---> Kubernetes --> Monitoring

=================================
Azure Policy --> Policy as a Code
=================================

This can be done through Code, They can deploy via Pipelines

Version Controlled Versioning of Policy