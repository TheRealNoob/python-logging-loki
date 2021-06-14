python-logging-loki
===================

[![PyPI version](https://img.shields.io/pypi/v/python-logging-loki.svg)](https://pypi.org/project/python-logging-loki/)
[![Python version](https://img.shields.io/badge/python-3.6%20%7C%203.7%20%7C%203.8-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/pypi/l/python-logging-loki.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://travis-ci.org/GreyZmeem/python-logging-loki.svg?branch=master)](https://travis-ci.org/GreyZmeem/python-logging-loki)

Python logging handler for Loki.  
https://grafana.com/loki

# Installation

```bash
pip install python-logging-loki
```

# Usage

```python
import logging
import logging_loki


handler = logging_loki.LokiHandler(
    url="https://my-loki-instance/loki/api/v1/push", 
    tags={"service": "my-app"},
)

logger = logging.getLogger("my-logger")
logger.addHandler(handler)
logger.error(
    "Something happened", 
    extra={"tags": {"host": "hostname"}},
)
```

Above example will send a log message of `Something happened` along with these labels:
- Default labels from handler
- Message level as `level`
- Logger's name as `logger`
- Labels from `tags` item of `extra` dict

# Auth

Loki has no native support for authentication.  Their expectation is that it should be handled by a middle layer such as Nginx.  See official [documentation](https://grafana.com/docs/loki/latest/operations/authentication/).

This lib has support for basic auth, but not for OAuth2.

```python
handler = logging_loki.LokiHandler(
    url="https://my-loki-instance/loki/api/v1/push",
    auth=("username", "password"),
)
```

# Kubernetes multitenancy

Promtail supports querying kubernetes for the name of the tenant (namespace) that your pod is running under.  This lib can't query that without kubectl and that would be a lot of code for this little library, but we can allow you to staticly define the tenant, and we will add that as a header. This would allow you to use the same process flow for promtail and this library.

```python
handler = logging_loki.LokiHandler(
    url="https://my-loki-instance/loki/api/v1/push",
    headers={"X-Scope-OrgID": "example-id"},
)
```

# Loki >= 0.4.0 support
This lib defaults to support for Loki version >= 0.4.0.  If you still use an earlier version, you can specify that support.

```python
handler = logging_loki.LokiHandler(
    url="https://my-loki-instance/api/prom/push",
    version = "0",
)
```

# Multithread support

The above example is blocking (i.e. each call will wait for the message to be sent).  
But you can use the built-in `QueueHandler` and` QueueListener` to send messages in a separate thread.  

```python
import logging.handlers
import logging_loki
from multiprocessing import Queue


queue = Queue(-1)
handler = logging.handlers.QueueHandler(queue)
handler_loki = logging_loki.LokiHandler(
    url="https://my-loki-instance/loki/api/v1/push", 
    tags={"application": "my-app"},
    auth=("username", "password"),
    version="1",
)
logging.handlers.QueueListener(queue, handler_loki)

logger = logging.getLogger("my-logger")
logger.addHandler(handler)
logger.error(...)
```

Or you can use `LokiQueueHandler` shortcut, which will automatically create listener and handler.

```python
import logging.handlers
import logging_loki
from multiprocessing import Queue


handler = logging_loki.LokiQueueHandler(
    Queue(-1),
    url="https://my-loki-instance/loki/api/v1/push", 
    tags={"application": "my-app"},
    auth=("username", "password"),
    version="1",
)

logger = logging.getLogger("my-logger")
logger.addHandler(handler)
logger.error(...)
```
