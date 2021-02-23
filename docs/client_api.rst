.. _client_api_label:

Client API
==========

*eProsima Micro XRCE-DDS* provides the user with a C API to create *eProsima Micro XRCE-DDS Clients* applications.
**All** functions needed to set up the *Client* can be found in the :code:`client.h` header.
That is the only header the user needs to include.

In this section, we provide the full API for the *Micro XRCE-DDS Client*.
As a nomenclature, this API uses the :code:`uxr_` prefix in all of its public functions and the :code:`uxr`
prefix in the types. In constants values, the :code:`UXR_` prefix is used.
The functions belonging to the public interface of the library are only those with the tag :code:`UXRDDLAPI`
in their declarations.

The functions are grouped as follows:

* :ref:`session_api`
* :ref:`entities_xml_api`
* :ref:`entities_ref_api`
* :ref:`entities_common_api`
* :ref:`read_access_api`
* :ref:`write_access_api`
* :ref:`discovery_api`
* :ref:`topic_serialization_api`
* :ref:`general_utilities_api`
* :ref:`transport_api`

.. _session_api:

Session
^^^^^^^

These functions are available even if no profile has been enabled.
The declaration of these function can be found in ``uxr/client/core/session/session.h``.

------

.. code-block:: c

    void uxr_init_session(uxrSession* session, uxrCommunication* comm, uint32_t key);

Initializes a session structure.
Once this function is called, a ``create_session`` call can be performed.


:session: Session structure where manage the session data.
:comm: Communication used for connecting to the *Agent*.
          All different transports have a common attribute uxrCommunication.
          This parameter can not be shared between active sessions.
:key: The key identifier of the *Client*.
         All *Clients* connected to an *Agent* must have a different key.

------

.. code-block:: c

    void uxr_set_status_callback(uxrSession* session, uxrOnStatusFunc on_status_func, void* args);

Assigns the callback for the *Agent* status messages.

:session: Session structure previously initialized.
:on_status_func: Function callback that will be called when a valid status message comes from the *Agent*.
:args: User pointer data.
       The args will be provided to ``on_status_func`` function.

------

.. code-block:: c

    void uxr_set_topic_callback(uxrSession* session, uxrOnTopicFunc on_topic_func, void* args);

Assigns the callback for topics.
The topics will be received only if a ``request_data`` function has been called.

:session: Session structure previously initialized.
:on_status_func: Function callback that will be called when a valid data message comes from the *Agent*.
:args: User pointer data.
       The args will be provided to ``on_topic_func`` function.

------

.. code-block:: c

    void uxr_set_request_callback(uxrSession* session, uxrOnRequestFunc on_request_func, void* args);

Sets the request callback. This will be called when the *Agent* sends a ``READ_DATA`` submessage associated with a ``Requester``.

:session: Session structure previously initialized.
:on_request_func: Function callback that will be called.
:args: User pointer data.
       The args will be provided to ``on_request_func`` function.

------

.. code-block:: c

    void uxr_set_reply_callback(uxrSession* session, uxrOnReplyFunc on_reply_func, void* args);

Sets the request callback. This will be called when the *Agent* sends a ``READ_DATA`` submessage associated with a ``Replier``.

:session: Session structure previously initialized.
:on_reply_func: Function callback that will be called.
:args: User pointer data.
       The args will be provided to ``on_reply_func`` function.

------

.. code-block:: c

    bool uxr_create_session(uxrSession* session);

Creates a new session with the *Agent*.
This function logs in a session, enabling any other XRCE communication with the *Agent*.

:session: Session structure previously initialized.

------

.. code-block:: c

    void uxr_create_session_retries(uxrSession* session, size_t retries);

Attempts to establish a new session on the *Agent* :code:`retries` times
This function logs in a session, enabling any other XRCE communication with the *Agent*.

:session: Session structure previously initialized.
:retries: Number of retries to try to create a session.

------

.. code-block:: c

    bool uxr_delete_session(uxrSession* session);

Deletes a session previously created.
All `XRCE` entities created with the session will be removed.
This function logs out a session, disabling any other `XRCE` communication with the *Agent*.

:session: Session structure previously initialized.

------

.. code-block:: c

    uxrStreamId uxr_create_output_best_effort_stream(uxrSession* session, uint8_t* buffer, size_t size);

Creates and initializes an output best-effort stream for writing.
The ``uxrStreamId`` returned represents the new stream and can be used to manage it.
The number of available calls to this function must be less or equal than ``CONFIG_MAX_OUTPUT_BEST_EFFORT_STREAMS`` CMake argument.

:session: Session structure previously initialized.
:buffer: Memory block where the messages will be written.
:size: Buffer size.

------

.. code-block:: c

    uxrStreamId uxr_create_output_reliable_stream(uxrSession* session, uint8_t* buffer, size_t size, size_t history);

Creates and initializes an output reliable stream for writing.
The ``uxrStreamId`` returned represents the new stream and can be used to manage it.
The number of available calls to this function must be less or equal than ``CONFIG_MAX_OUTPUT_RELIABLE_STREAMS`` CMake argument.

:session: Session structure previously initialized.
:buffer: Memory block where the messages will be written.
:size: Buffer size.
:history: History used for reliable connection.
          The buffer size will be split into smaller buffers using this value.
          The history must be a power of two.

------

.. code-block:: c

    uxrStreamId uxr_create_input_best_effort_stream(uxrSession* session);

Creates and initializes an input best-effort stream for receiving messages.
The ``uxrStreamId`` returned represents the new stream and can be used to manage it.
The number of available calls to this function must be less or equal than ``CONFIG_MAX_INPUT_BEST_EFFORT_STREAMS`` CMake argument.

:session: Session structure previously initialized.

------

.. code-block:: c

    uxrStreamId uxr_create_input_reliable_stream(uxrSession* session, uint8_t* buffer, size_t size, size_t history);

Creates and initializes an input reliable stream for receiving messages.
The returned ``uxrStreamId`` represents the new stream and can be used to manage it.
The number of available calls to this function must be less or equal than ``CONFIG_MAX_INPUT_RELIABLE_STREAMS`` CMake argument.

:session: Session structure previously initialized.
:buffer: Memory block where the messages will be storaged.
:size: Buffer size.
:history: History used for reliable connection.
          The buffer will be split into smaller buffers using this value.
          The history must be a power of two.

------

.. code-block:: c

    void uxr_flash_output_streams(uxrSession* session);

Flashes all output streams sending the data through the transport.

:session: Session structure previously initialized.

------

.. code-block:: c

    void uxr_run_session_time(uxrSession* session, int time);

This function processes the internal functionality of a session.
It implies:

1. Flushing all output streams sending the data through the transport.
2. If there is any reliable stream, it will perform the associated reliable behaviour to ensure communication.
3. Listening messages from the *Agent* and calling the associated callback if it exists (a topic callback or a status callback).

The ``time`` suffix function version will perform these actions and will listen to messages for a ``time`` duration.
Only when the time waiting for a message overcome the ``time`` duration, the function finishes.
The function will return ``true`` if the sending data have been confirmed, ``false`` otherwise.

:session: Session structure previously initialized.
:time: Time for waiting, in milliseconds.
          For waiting without timeout, set the value to ``UXR_TIMEOUT_INF``

------

.. code-block:: c

    void uxr_run_session_until_timeout(uxrSession* session, int timeout);

This function processes the internal functionality of a session.
It implies:

1. Flushing all output streams sending the data through the transport.
2. If there is any reliable stream, it will perform the associated reliable behaviour to ensure communication.
3. Listening messages from the *Agent* and call the associated callback if it exists (a topic callback or a status callback).

The ``_until_timeout`` suffix function version will perform these actions until receiving one message.
Once the message has been received or the timeout has been reached, the function finishes.
Only when the time waiting for a message overcome the ``timeout`` duration, the function finishes.
The function will return ``true`` if it has received a message, ``false`` if the timeout has been reached.

:session: Session structure previously initialized.
:timeout: Time for waiting for a new message, in milliseconds.
          For waiting without timeout, set the value to ``UXR_TIMEOUT_INF``

------

.. code-block:: c

    bool uxr_run_session_until_confirm_delivery(uxrSession* session, int timeout);

This function processes the internal functionality of a session.
It implies:

1. Flushing all output streams sending the data through the transport.
2. If there is any reliable stream, it will perform the associated reliable behaviour to ensure communication.
3. Listening messages from the *Agent* and call the associated callback if it exists (a topic callback or a status callback).

The ``_until_confirm_delivery`` suffix function version will perform these actions during ``timeout``
or until the output reliable streams confirm that the sent messages have been received by the *Agent*.
The function will return ``true`` if the sent data have been confirmed, ``false`` otherwise.

:session: Session structure previously initialized.
:timeout: Maximum waiting time for a new message, in milliseconds.
          For waiting without timeout, set the value to ``UXR_TIMEOUT_INF``

------

.. code-block:: c

    bool uxr_run_session_until_all_status(uxrSession* session, int timeout, const uint16_t* request_list,
                                          uint8_t* status_list, size_t list_size);

This function processes the internal functionality of a session.
It implies:

1. Flushing all output streams sending the data through the transport.
2. If there is any reliable stream, it will perform the associated reliable behaviour to ensure communication.
3. Listening messages from the *Agent* and call the associated callback if it exists (a topic callback or a status callback).

The ``_until_all_status`` suffix function version will perform these actions during ``timeout`` duration
or until all requested status had been received.
The function will return ``true`` if all status have been received and all of them have the value ``UXR_STATUS_OK`` or ``UXR_STATUS_OK_MATCHED``, ``false`` otherwise.

:session: Session structure previously initialized.
:timeout: Maximum waiting time for a new message, in milliseconds.
          For waiting without timeout, set the value to ``UXR_TIMEOUT_INF``
:request_list: An array of requests to confirm with a status.
:status_list: An uninitialized array with the same size as ``request_list`` where the status values will be written.
              The position of status in the list corresponds to the request at the same position in ``request_list``.
:list_size: The size of ``request_list`` and ``status_list`` arrays.

------

.. code-block:: c

    bool uxr_run_session_until_one_status(uxrSession* session, int timeout, const uint16_t* request_list,
                                          uint8_t* status_list, size_t list_size);

This function processes the internal functionality of a session.
It implies:

1. Flushing all output streams sending the data through the transport.
2. If there is any reliable stream, it will perform the associated reliable behaviour to ensure communication.
3. Listening messages from the *Agent* and call the associated callback if it exists (a topic callback or a status callback).

The ``_until_one_status`` suffix function version will perform these actions during ``timeout`` duration
or until one requested status had been received.
The function will return ``true`` if one status have been received and has the value ``UXR_STATUS_OK`` or ``UXR_STATUS_OK_MATCHED``, ``false`` otherwise.

:session: Session structure previously initialized.
:timeout: Maximum waiting time for a new message, in milliseconds.
          For waiting without timeout, set the value to ``UXR_TIMEOUT_INF``
:request_list: An array of requests that can be confirmed.
:status_list: An uninitialized array with the same size as ``request_list`` where the status value will be written.
              The position of the status in the list corresponds to the request at the same position in ``request_list``.
:list_size: The size of ``request_list`` and ``status_list`` arrays.

------

.. code-block:: c

    bool uxr_run_session_until_data(uxrSession* session, int timeout);

This function processes the internal functionality of a session.
It implies:

1. Flushing all output streams sending the data through the transport.
2. If there is any reliable stream, it will operate according to the associated reliable behaviour to ensure communication.
3. Listening messages from the *Agent* and calling the associated callback if it exists (a topic callback or a status callback).

The ``_until_data`` suffix function version will perform these actions during a ``timeout`` duration
or until a subscription data, request or reply is received.
The function will return ``true`` if a subscription data, request or reply is received, and ``false`` otherwise.

:session: Session structure previously initialized.
:timeout: Maximum waiting time for a new message, in milliseconds.
          For waiting without timeout, set the value to ``UXR_TIMEOUT_INF``

------

.. code-block:: c

    bool uxr_sync_session(uxrSession* session, int timeout);

This function synchronizes the session time with the *Agent* using the NTP protocol by default.

:session: Session structure previously initialized.
:timeout: The waiting time in milliseconds.

------

.. code-block:: c

    int64_t uxr_epoch_millis(uxrSession* session);

This function returns the epoch time in milliseconds taking into account the offset computed during the time synchronization.

:session: Session structure previously initialized.

------

.. code-block:: c

    int64_t uxr_epoch_nanos(uxrSession* session);

This function returns the epoch time in nanoseconds taking into account the offset computed during the time synchronization.

:session: Session structure previously initialized.

------

.. _entities_xml_api:

Create entities by XML
^^^^^^^^^^^^^^^^^^^^^^

These functions are enabled when ``PROFILE_CREATE_ENTITIES_XML`` is activated as a CMake argument.
The declaration of these functions can be found in ``uxr/client/profile/session/create_entities_xml.h``.

------

.. code-block:: c

    uint16_t uxr_buffer_create_participant_xml(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                               uint16_t domain, const char* xml, uint8_t mode);

Creates a `participant` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_PARTICIPANT_ID``.
:xml: An XML representation of the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_topic_xml(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                         uxrObjectId participant_id, const char* xml, uint8_t mode);

Creates a `topic` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_TOPIC_ID``.
:participant_id: The identifier of the associated participant.
            The type must be ``UXR_PARTICIPANT_ID``.
:xml: An XML representation of the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_publisher_xml(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                             uxrObjectId participant_id, const char* xml, uint8_t mode);

Creates a `publisher` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_PUBLISHER_ID``.
:participant_id: The identifier of the associated participant.
            The type must be ``UXR_PARTICIPANT_ID``.
:xml: An XML representation of the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_subscriber_xml(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                              uxrObjectId participant_id, const char* xml, uint8_t mode);

Creates a `subscriber` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_SUBSCRIBER_ID``.
:participant_id: The identifier of the associated participant.
            The type must be ``UXR_PARTICIPANT_ID``.
:xml: An XML representation of the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_datawriter_xml(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                              uxrObjectId publisher_id, const char* xml, uint8_t mode);

Creates a `datawriter` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_DATAWRITER_ID``.
:publisher_id: The identifier of the associated publisher.
            The type must be ``UXR_PUBLISHER_ID``.
:xml: An XML representation of the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_datareader_xml(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                              uxrObjectId subscriber_id, const char* xml, uint8_t mode);

Creates a `datareader` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_DATAREADER_ID``.
:subscriber_id: The identifier of the associated subscriber.
            The type must be ``UXR_SUBSCRIBER_ID``.
:xml: An XML representation of the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_requester_xml(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                             uxrObjectId participant_id, const char* xml, uint8_t mode);

Creates a `requester` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message it is necessary to call the ``uxr_flash_output_streams`` or ``uxr_run_session`` functions.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_REQUESTER_ID``.
:participant_id: The identifier of the associated participant.
            The type must be ``UXR_PARTICIPANT_ID``.
:xml: An XML representation of the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_replier_xml(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                           uxrObjectId participant_id, const char* xml, uint8_t mode);

Creates a `replier` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message it is necessary to call the ``uxr_flash_output_streams`` or ``uxr_run_session`` functions.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_REPLIER_ID``.
:participant_id: The identifier of the associated participant.
            The type must be ``UXR_PARTICIPANT_ID``.
:xml: An XML representation of the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. _entities_ref_api:

Create entities by reference
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These functions are enabled when ``PROFILE_CREATE_ENTITIES_REF`` is activated as a CMake argument.
The declaration of these functions can be found in ``uxr/client/profile/session/create_entities_ref.h``.

------

.. code-block:: c

    uint16_t uxr_buffer_create_participant_ref(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                               const char* ref, uint8_t mode);

Creates a `participant` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_PARTICIPANT_ID``
:ref: A reference to the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_topic_ref(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                         uxrObjectId participant_id, const char* ref, uint8_t mode);

Creates a `topic` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_TOPIC_ID``
:participant_id: The identifier of the associated participant.
            The type must be ``UXR_PARTICIPANT_ID``
:ref: A reference to the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_datawriter_ref(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                              uxrObjectId publisher_id, const char* ref, uint8_t mode);

Creates a `datawriter` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_DATAWRITER_ID``
:publisher_id: The identifier of the associated publisher.
            The type must be ``UXR_PUBLISHER_ID``
:ref: A reference to the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_datareader_ref(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                              uxrObjectId subscriber_id, const char* ref, uint8_t mode);

Creates a `datareader` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_DATAREADER_ID``.
:subscriber_id: The identifier of the associated subscriber.
            The type must be ``UXR_SUBSCRIBER_ID``.
:ref: A reference to the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_requester_ref(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                             uxrObjectId participant_id, const char* ref, uint8_t mode);

Creates a `requester` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message it is necessary to call the ``uxr_flash_output_streams`` or ``uxr_run_session`` functions.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_REQUESTER_ID``.
:participant_id: The identifier of the associated participant.
            The type must be ``UXR_PARTICIPANT_ID``.
:ref: A reference to the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. code-block:: c

    uint16_t uxr_buffer_create_replier_ref(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id,
                                           uxrObjectId participant_id, const char* ref, uint8_t mode);

Creates a `replier` entity in the *Agent*.
The message is only written into the stream buffer.
To send the message it is necessary to call the ``uxr_flash_output_streams`` or ``uxr_run_session`` functions.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the new entity.
            Later, the entity can be referenced with this id.
            The type must be ``UXR_REPLIER_ID``.
:participant_id: The identifier of the associated participant.
            The type must be ``UXR_PARTICIPANT_ID``.
:ref: A reference to the new entity.
:mode: Determines the creation entity mode.
        The Creation Mode Table describes the entities' creation behaviour according to the ``UXR_REUSE`` and ``UXR_REPLACE`` flags.

------

.. _entities_common_api:

Create entities common profile
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These functions are enabled when ``PROFILE_CREATE_ENTITIES_XML`` or ``PROFILE_CREATE_ENTITIES_REF`` are activated as CMake arguments.
The declaration of these functions can be found in ``uxr/client/profile/session/common_create_entities.h``.

------

.. code-block:: c

    uint16_t uxr_buffer_delete_entity(uxrSession* session, uxrStreamId stream_id, uxrObjectId object_id);

Removes an entity.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The identifier of the object which will be deleted.

------

.. _read_access_api:

Read access profile
^^^^^^^^^^^^^^^^^^^

These functions are enabled when ``PROFILE_READ_ACCESS`` is activated as a CMake argument.
The declaration of these functions can be found in ``uxr/client/profile/session/read_access.h``.

------

.. code-block:: c

    uint16_t uxr_buffer_request_data(uxrSession* session, uxrStreamId stream_id, uxrObjectId datareader_id,
                                     uxrStreamId data_stream_id, uxrDeliveryControl* delivery_control);

This function requests a read from a `datareader` of the *Agent*.
The returned value is an identifier of the request.
All received topic will have the same request identifier.
The topics will be received at the callback topic through the ``run_session`` function.
If there is no error with the request data, the topics will be received generating a status callback with the value ``UXR_STATUS_OK``.
If there is an error, a status error will be sent by the *Agent*.
The message is only written into the stream buffer.
To send the message is necessary call to ``uxr_flash_output_streams`` or to ``uxr_run_session`` function.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:object_id: The `datareader` ID that will read the topic from the DDS World.
:data_stream_id: The input stream ID where the data will be received.
:delivery_control: Optional information about how the delivery must be.
                   A ``NULL`` value is accepted, in this case, only one topic will be received.

------

.. _write_access_api:

Write access profile
````````````````````
These functions are enabled when ``PROFILE_WRITE_ACCESS`` is activated as a CMake argument.
The declaration of these functions can be found in ``uxr/client/profile/session/write_access.h``.

------

.. code-block:: c

    bool uxr_prepare_output_stream(uxrSession* session, uxrStreamId stream_id, uxrObjectId datawriter_id,
                                   struct ucdrBuffer* ub_topic, uint32_t topic_size);

Requests a writing into a specific output stream.
This function will initialize an ``ucdrBuffer`` struct where a topic of ``topic_size`` size must be serialized.
Whether the necessary gap for writing a ``topic_size`` bytes into the stream, the returned value is ``true``, otherwise ``false``.
The topic will be sent in the next ``run_session`` function.

NOTE: All ``topic_size`` bytes requested will be sent to the *Agent* after a ``run_session`` call, no matter if the ``ucdrBuffer`` has been used or not.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:datawriter_id: The `datawriter` ID that will write the topic to the DDS World.
:ub_topic: An ``ucdrBuffer`` struct used to serialize the topic.
           This struct points to a requested gap into the stream.
:topic_size: The bytes that will be reserved in the stream.

------

.. code-block:: c

    bool uxr_buffer_request(uxrSession* session, uxrStreamId stream_id, uxrObjectId requester_id, uint8_t* buffer, size_t len);

Buffers a request into a specific output stream.
The request will be sent in the next ``run_session`` function call.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:requester_id: The `requester`'s ID that will write the request to the DDS World.
:buffer: The raw buffer that contains the serialized request.
:len: The size of the serialized request.

------

.. code-block:: c

    bool uxr_buffer_reply(uxrSession* session, uxrStreamId stream_id, uxrObjectId replier_id, uint8_t* buffer, size_t len);

Buffers a reply into a specific output stream.
The request will be sent in the next ``run_session`` function call.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:replier_id: The `replier`'s ID that will write the reply to the DDS World.
:buffer: The raw buffer that contains the serialized reply.
:len: The size of the serialized reply.

------

.. code-block:: c

    bool uxr_prepare_output_stream_fragmented(uxrSession* session, uxrStreamId stream_id, uxrObjectId datawriter_id,
                                              struct ucdrBuffer* ub, size_t topic_size, uxrOnBuffersFull flush_callback);

Requests to allocate an output stream of ``topic_size`` bytes for a write operation.
This function will initialize an ``ucdrBuffer`` struct where a topic of ``topic_size`` size will be serialized.
If there is sufficient space for writing ``topic_size`` bytes into the reliable stream, the returned value is ``true``, otherwise it is ``false``.
The topic will be sent in the following ``run_session`` function. If during the serialization process the buffer gets overfilled, the ``flush_callback`` function will be called and the user will be in charge of running a session for flushing the stream.

NOTE: This approach is not valid with best-effort streams.

:session: Session structure previously initialized.
:stream_id: The output stream ID where the message will be written.
:datawriter_id: The `datawriter` ID that will write the topic to the DDS World.
:ub: An ``ucdrBuffer`` struct used to serialize the topic.
           This struct points to a requested memory slot in the stream.
:topic_size: The slot, in bytes, that will be reserved in the stream.
:flush_callback: Callback for flushing the output buffers.

------

.. _discovery_api:

Discovery profile
^^^^^^^^^^^^^^^^^

The discovery profile allows discovering *Agents* in the network by UDP.
The reachable *Agents* will respond to the discovery call sending information about them, as their IP and port.
There are two modes: multicast and unicast.
The discovery phase can be performed before the `uxr_create_session` call to determine the *Agent* to connect with.
These functions are enabled when ``PROFILE_DISCOVERY`` is activated as a CMake argument.
The declaration of these functions can be found in ``uxr/client/profile/discovery/discovery.h``.

*This feature is only available on Linux.*

------

.. code-block:: c

    bool uxr_discovery_agents_multicast(uint32_t attempts, int period,
                                        uxrOnAgentFound on_agent_func, void* args, uxrAgentAddress* chosen);

Searches into the network using multicast IP "239.255.0.2" and port 7400 (default used by the *Agent*) to discover *Agents*.

:attempts: The number of attempts to send the discovery message to the network.
:period: How will often be sent the discovery message to the network.
:on_agent_func: The callback function that will be called when an *Agent* was discovered.
                The callback returns a boolean value.
                A `true` means that the discovery routine will be finished.
                The current *Agent* will be selected as *chosen*.
                A `false` implies that the discovery routine must continue searching *Agents*.
:args: User arguments passed to the callback function.
:chosen: If the callback function was returned `true`, this value will contain the *Agent* value of the callback.

------

.. code-block:: c

    bool uxr_discovery_agents_unicast(uint32_t attempts, int period,
                                      uxrOnAgentFound on_agent_func, void* args, uxrAgentAddress* chosen,
                                      const uxrAgentAddress* agent_list, size_t agent_list_size);

Searches into the network using a list of unicast directions to discover *Agents*.

:attempts: The number of attempts to send the discovery message to the network.
:period: How will often be sent the discovery message to the network.
:on_agent_func: The callback function that will be called when an *Agent* is discovered.
                The callback returns a boolean value.
                A ``true`` means that the discovery routine will be finished.
                The current *Agent* will be selected as *chosen*.
                A ``false`` implies that the discovery routine must continue searching *Agents*.
:args: User arguments passed to the callback function.
:chosen: If the callback function was returned ``true``, this value will contain the *Agent* value of the callback.
:agent_list: The list of addresses where discover *Agent*.
             By default, the *Agents* will be listened at **port 7400** the discovery messages.
:agent_list_size: The size of the ``agent_list``.

------

.. _topic_serialization_api:

Topic serialization
^^^^^^^^^^^^^^^^^^^

Functions to serialize and deserialize topics.
These functions are generated automatically by *eProsima Micro XRCE-DDS Gen* utility over an IDL file with a topic `TOPICTYPE`.
The declaration of these functions can be found in the generated file ``TOPICTYPE.h``.

------

.. code-block:: c

    bool TOPICTYPE_serialize_topic(struct ucdrBuffer* writer, const TOPICTYPE* topic);

Serializes a topic into an ``ucdrBuffer``.
The returned value indicates if the serialization was successful.

:writer: An ``ucdrBuffer`` representing the buffer for the serialization.
:topic: Struct to serialize.

------

.. code-block:: c

    bool TOPICTYPE_deserialize_topic(struct ucdrBuffer* reader, TOPICTYPE* topic);

Deserializes a topic from an ucdrBuffer.
The returned value indicates if the serialization was successful.

:reader: An ucdrBuffer representing the buffer for the deserialization.
:topic: Struct where deserialize.

------

.. code-block:: c

    uint32_t TOPICTYPE_size_of_topic(const TOPICTYPE* topic, uint32_t size);

Counts the number of bytes that the topic will need in an `ucdrBuffer`.

:topic: Struct to count the size.
:size: Number of bytes already written into the `ucdrBuffer`.
       Typically, its value is `0` if the purpose is to use in ``uxr_prepare_output_stream`` function.

------

.. _general_utilities_api:

General utilities
^^^^^^^^^^^^^^^^^

Utility functions.
The declaration of these functions can be found in ``uxr/client/core/session/stream_id.h`` and ``uxr/client/core/session/object_id.h``.

------

.. code-block:: c

    uxrStreamId uxr_stream_id(uint8_t index, uxrStreamType type, uxrStreamDirection direction);

Creates a stream identifier.
This function does not create a new stream, only creates its identifier to be used in the *Client* API.

:index: Identifier of the stream, its value corresponding to the creation order of the stream, different for each `type`.
:type: The type of the stream, it can be ``UXR_BEST_EFFORT_STREAM`` or ``UXR_RELIABLE_STREAM``.
:direction: Represents the direction of the stream. It can be ``UXR_INPUT_STREAM`` or ``UXR_OUTPUT_STREAM``.

------

.. code-block:: c

    uxrStreamId uxr_stream_id_from_raw(uint8_t stream_id_raw, uxrStreamDirection direction);

Creates a stream identifier.
This function does not create a new stream, only creates its identifier to be used in the *Client* API.

:stream_id_raw: Identifier of the stream.
      It goes from 0 to 255.
      0 is for internal library use.
      1 to 127, for best effort.
      128 to 255, for reliable.
:direction: Represents the direction of the stream. It can be ``UXR_INPUT_STREAM`` or ``MT_OUTPUT_STREAM``.

------

.. code-block:: c

    uxrObjectId uxr_object_id(uint16_t id, uint8_t type);

Creates an identifier for reference an entity.

:id: Identifier of the object, different for each `type`
     (can be several IDs with the same ID if they have different types).
:type: The type of the entity.
       It can be: ``UXR_PARTICIPANT_ID``, ``UXR_TOPIC_ID``, ``UXR_PUBLISHER_ID``, ``UXR_SUBSCRIBER_ID``, ``UXR_DATAWRITER_ID`` or ``UXR_DATAREADER_ID``.

------

.. _transport_api:

Transport
^^^^^^^^^

These functions are platform dependent.
The declaration of these functions can be found in the ``uxr/client/profile/transport/`` folder.
The common init transport functions follow the nomenclature below.

.. TODO: new functions have been added. Should the description above be improved and expanded?
(ping, custom transport..)

------

.. code-block:: c

    bool uxr_init_udp_transport(uxrUDPTransport* transport, const char* ip, uint16_t port);

Initializes a UDP connection.

:transport: The uninitialized structure used for managing the transport.
            This structure must be accessible during the connection.
:ip: *Agent* IP.
:port: *Agent* port.

------

.. code-block:: c

    bool uxr_init_tcp_transport(uxrTCPTransport* transport, const char* ip, uint16_t port);

Initializes a TCP connection.
If the TCP is used, the behaviour of best-effort streams will be similar to reliable streams in UDP.

:transport: The uninitialized structure used for managing the transport.
            This structure must be accessible during the connection.
:ip: *Agent* IP.
:port: *Agent* port.

------

.. code-block:: c

    bool uxr_init_serial_transport(uxrSerialTransport* transport, const int fd, uint8_t remote_addr, uint8_t local_addr);

Initializes a Serial connection using a file descriptor

:transport: The uninitialized structure used for managing the transport.
            This structure must be accessible during the connection.
:fd: File descriptor of the serial connection. Usually, the `fd` comes from the ``open`` OS function.
:remote_addr: Identifier of the *Agent* in the serial connection.
              By default, the *Agent* identifier in a serial is 0.
:local_addr: Identifier of the *Client* in the serial connection.

------

.. code-block:: c

    bool uxr_close_PROTOCOL_transport(PROTOCOLTransport* transport);

Closes a transport previously opened. `PROTOCOL` can be ``udp``, ``tcp`` or ``serial``.

:transport: The transport to close.

------

.. code-block:: c

    bool uxr_ping_agent(const uxrCommunication* comm, const int timeout);

[TODO]

:comm: [TODO]
:timeout: [TODO]

------

.. code-block:: c

    bool uxr_ping_agent_attempts(const uxrCommunication* comm, const int timeout, const uint8_t attempts);

[TODO]

:comm: [TODO]
:timeout: [TODO]
:attempts: [TODO]

------

.. code-block:: c

    void uxr_set_custom_transport_callbacks(uxrCustomTransport* transport, bool framing, open_custom_func open,
                                            close_custom_func close, write_custom_func write, read_custom_func read);

Assigns the callback for custom transport.

:transport: The uninitialized structure used for managing the transport.
            This structure must be accessible during the connection.
:framing: Enables or disables Stream Framing Protocol for a custom transport.
:open: Callback for opening a custom transport.
:close: Callback for closing a custom transport.
:write: Callback for writing to a custom transport.
:read: Callback for reading from a custom transport.

The function signatures for the above callbacks are:

.. code-block:: c

    typedef bool (*open_custom_func)(struct uxrCustomTransport*);

Where ``struct uxrCustomTransport*`` will have the ``args`` passed through ``bool uxr_init_custom_transport(uxrCustomTransport* transport, void * args);``

.. code-block:: c

    typedef bool (*close_custom_func)(struct uxrCustomTransport*);

Where ``struct uxrCustomTransport*`` will have the ``args`` passed through ``bool uxr_init_custom_transport(uxrCustomTransport* transport, void * args);``

.. code-block:: c

    typedef size_t (*write_custom_func)(struct uxrCustomTransport*, const uint8_t*, size_t, uint8_t*);

Where ``struct uxrCustomTransport*`` refers to the opened transport structure,  the first ``const uint8_t*`` is the buffer to be sent, ``size_t`` is the length of the buffer, and the last ``uint8_t*`` is an error code that should be set when the write process has any problem. This function should return the number of bytes sent successfully.

.. code-block:: c

    typedef size_t (*read_custom_func)(struct uxrCustomTransport*, uint8_t*, size_t, int, uint8_t*);

Where ``struct uxrCustomTransport*`` refers to the opened transport structure,  the first ``uint8_t*`` is the buffer to be write with the received bytes, following ``size_t`` is the length of the buffer, the following ``int`` is the maximum time in milliseconds that the read operating should take and the last ``uint8_t*`` is an error code that should be set when the read process has any problem. This function should return the number of bytes received successfully.