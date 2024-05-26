Tutorial
--------

Parse models
============

Pydantic :func:`~pydantic.BaseModel.model_validate` method can be used to parse and validate SCIM2 payloads:

.. code-block:: python

    from pydantic_scim2 import User
    import datetime

    payload = {
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
        "id": "2819c223-7f76-453a-919d-413861904646",
        "userName": "bjensen@example.com",
        "meta": {
            "resourceType": "User",
            "created": "2010-01-23T04:56:22Z",
            "lastModified": "2011-05-13T04:42:34Z",
            "version": 'W\\/"3694e05e9dff590"',
            "location": "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646",
        },
    }

    user = User.model_validate(payload)
    assert user.user_name == "bjensen@example.com"
    assert user.meta.created == datetime.datetime(
        2010, 1, 23, 4, 56, 22, tzinfo=datetime.timezone.utc
    )

Serialize models
================

Pydantic :func:`~pydantic.BaseModel.model_dump` method can be used to produce valid SCIM2 payloads:

.. code-block:: python

    from pydantic_scim2 import User, Meta
    import datetime

    user = User(
        id="2819c223-7f76-453a-919d-413861904646",
        user_name="bjensen@example.com",
        meta=Meta(
            resource_type="User",
            created=datetime.datetime(2010, 1, 23, 4, 56, 22, tzinfo=datetime.timezone.utc),
            last_modified=datetime.datetime(2011, 5, 13, 4, 42, 34, tzinfo=datetime.timezone.utc),
            version='W\\/"3694e05e9dff590"',
            location="https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646",
        ),
    )
    dump = user.model_dump(exclude_none=True, by_alias=True, mode="json")
    assert dump == {
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
        "id": "2819c223-7f76-453a-919d-413861904646",
        "userName": "bjensen@example.com",
        "meta": {
            "resourceType": "User",
            "created": "2010-01-23T04:56:22Z",
            "lastModified": "2011-05-13T04:42:34Z",
            "version": 'W\\/"3694e05e9dff590"',
            "location": "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646",
        },
    }


Typed ListResponse
==================

:class:`~pydantic_scim2.ListResponse` models take a type or a :data:`~typing.Union` of types.
You must pass the type you expect in the response, e.g. :class:`~pydantic_scim2.ListResponse[User]` or :class:`~pydantic_scim2.ListResponse[Union[User, Group]]`.
If a response resource type cannot be found, a `pydantic.ValidationError` will be raised.

.. code-block:: python

    from typing import Union
    from pydantic_scim2 import User, Group, ListResponse

    payload = {
        "totalResults": 2,
        "itemsPerPage": 10,
        "startIndex": 1,
        "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
        "Resources": [
            {
                "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
                "id": "2819c223-7f76-453a-919d-413861904646",
                "userName": "bjensen@example.com",
                "meta": {
                    "resourceType": "User",
                    "created": "2010-01-23T04:56:22Z",
                    "lastModified": "2011-05-13T04:42:34Z",
                    "version": 'W\\/"3694e05e9dff590"',
                    "location": "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646",
                },
            },
            {
                "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
                "id": "e9e30dba-f08f-4109-8486-d5c6a331660a",
                "displayName": "Tour Guides",
                "members": [
                    {
                        "value": "2819c223-7f76-453a-919d-413861904646",
                        "$ref": "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646",
                        "display": "Babs Jensen",
                    },
                    {
                        "value": "902c246b-6245-4190-8e05-00816be7344a",
                        "$ref": "https://example.com/v2/Users/902c246b-6245-4190-8e05-00816be7344a",
                        "display": "Mandy Pepperidge",
                    },
                ],
                "meta": {
                    "resourceType": "Group",
                    "created": "2010-01-23T04:56:22Z",
                    "lastModified": "2011-05-13T04:42:34Z",
                    "version": 'W\\/"3694e05e9dff592"',
                    "location": "https://example.com/v2/Groups/e9e30dba-f08f-4109-8486-d5c6a331660a",
                },
            },
        ],
    }

    response = ListResponse[Union[User, Group]].model_validate(payload)
    user, group = response.resources
    assert isinstance(user, User)
    assert isinstance(group, Group)


Custom models
=============

You can write your own model and use it the same way than the other pydantic-scim2 models. Just inherit from `:class:`pydantic_scim2.Resource`:

.. code-block:: python

    from pydantic_scim2 import Resource
    from enum import Enum

    class Pet(Resource):
        class Type(str, Enum):
            dog = "dog"
            cat = "cat"

        name : str
        """The name of the pet."""

        type: Type
        """The pet type."""
