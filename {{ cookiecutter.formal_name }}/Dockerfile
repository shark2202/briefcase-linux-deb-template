FROM {{ cookiecutter.docker_base_image }}

# Set the working directory
WORKDIR /app

# Make sure installation of tzdata is non-interactive
ENV DEBIAN_FRONTEND="noninteractive" \
# Disable pip warnings about running as root (pip >= 22.1)
    PIP_ROOT_USER_ACTION="ignore"
{% if cookiecutter.python_source == "deadsnakes" %}
# Install the deadsnakes PPA so we can get arbitrary Python versions
RUN apt-get update -y && \
    apt-get install --no-install-recommends -y software-properties-common && \
    apt-add-repository ppa:deadsnakes/ppa

# Install Python from deadsnakes
RUN apt-get update -y && \
    apt-get install --no-install-recommends -y \
        python{{ cookiecutter.python_version|py_tag }}-dev \
        python{{ cookiecutter.python_version|py_tag }}-venv
{% elif cookiecutter.python_source == "system" %}
# Install System python
RUN apt-get update -y && \
    apt-get install --no-install-recommends -y \
        python{{ cookiecutter.python_version|py_tag }}-dev \
        python{{ cookiecutter.python_version|py_tag }}-venv \
        python{{ cookiecutter.python_version|py_tag }}-pip
{% endif %}
# Ensure Docker user UID:GID matches host user UID:GID (beeware/briefcase#403)
# Use --non-unique to avoid problems when the UID:GID of the host user
# collides with entries provided by the Docker container.
ARG HOST_UID
ARG HOST_GID
RUN groupadd --non-unique --gid $HOST_GID briefcase && \
    useradd --non-unique --uid $HOST_UID --gid $HOST_GID brutus --home /home/brutus && \
    mkdir -p /home/brutus && chown brutus:briefcase /home/brutus

# As root, Install system packages required by app
USER root
ARG SYSTEM_REQUIRES
RUN apt-get install --no-install-recommends -y ${SYSTEM_REQUIRES}

# Use the brutus user for operations in the container
USER brutus

{% if cookiecutter.python_source == "deadsnakes" %}
# Ensure pip is available; do this as the brutus user
# to ensure that the pip cache is created user-readable.
RUN python{{ cookiecutter.python_version|py_tag }} -m ensurepip
{% endif %}

# ========== START USER PROVIDED CONTENT ==========
{{ cookiecutter.dockerfile_extra_content }}
