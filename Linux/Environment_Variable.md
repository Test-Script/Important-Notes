==================================================================
                Linux Environment Variables
==================================================================


Local --> NAME="Gemini" --> Only this specific terminal window.

Exported --> export NAME="Gemini" --> This window + any apps/scripts started from it.

Permanent --> (Added to .bashrc) --> All future terminal sessions for that user.

=================================================================
How to test the scope yourself
=================================================================

In your terminal, type: FOO="Local"

Then type: bash (this starts a "child" shell inside your current one).

Type: echo $FOO — It will be empty.

Type exit to go back to the parent, then try: export FOO="Global"

Type bash again, then echo $FOO — It will now say "Global".

=================================================================
ConfigMap & Secret in Kubernetes
=================================================================

When you move from a single Linux machine to Kubernetes (K8s), the concept of environment variables stays the same, but the delivery mechanism changes.

Instead of manually typing export or editing a .bashrc file on a disk, Kubernetes uses ConfigMaps and Secrets to "inject" those variables into your containers (your Linux environments) at runtime.

1. ConfigMap: The Public Settings

    A ConfigMap is used for non-sensitive configuration—stuff you wouldn't mind a teammate seeing.

    How it works: You define a list of key-value pairs in a YAML file.

    The Injection: When your Pod (container) starts, Kubernetes looks at the ConfigMap and "pours" those values into the container's environment.

    Example: Storing a database URL like DB_HOST=prod-db.example.com.


2. Secret: The Locked Vault

    Secrets are identical to ConfigMaps in function, but they are handled with extra care by Kubernetes.

    How it works: Data is stored in a way that prevents it from being accidentally logged or shown in plain text.

    The Injection: Just like a ConfigMap, these are injected as environment variables when the container starts.

    Example: Storing a DB_PASSWORD=SuperSecret123.

================================================================

The "Pod" Perspective

To the application running inside the container, it doesn't know it's in Kubernetes. When your code calls os.getenv("DB_HOST") (in Python) or process.env.DB_HOST (in Node.js), it just sees a standard Linux environment variable. Kubernetes simply acted as the "automated hand" that set those variables before your program started.