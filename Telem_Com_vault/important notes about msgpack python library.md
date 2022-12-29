These notes are from repository of official python implementation msgpack package. you can find the repository [here](https://github.com/msgpack/msgpack-python).
- Use `use_bin_type=False`(default=True for later versions) to encode data according to the old format of msgpack standard. For decoding a message with old format, use `raw=True`(default=False for later versions)(decode str(raw) -> python bytes).
- `strict_map_key=False` (default=True) for using arbitrary type for dictionary keys. disabled by default for security issues.
- `max_buffer_size` is by default 100 MiB. for security, you should consider it.
- `packb`and its ailias `dumps` encodes data and `unpackb` and its ailias `loads` decodes the data. These ailiases are for compatibility with json and pickle packages.
- To decode list data, always use `use_list` (I think with False value) for backward compatibility(I think it means to be able to decode the encoded data with older versions). when `use_list=False`, tuples will be used instead of list for decoding. tuples are lighter and are better for performance critical cases.
- Use `Unpacker` method for decoding a buffer(stream) of multiple encoded data. for buffer you can use `BytesIO`.
- for de/serialization of custom data types you should write the decode/encode functions for that type and for encoding set `default` and for decoding set `object_hook` to the corresponding functions. these functions actually convert the data to a supported type of data for encoding and in the case of decoding check for example for a specific field/key in the message and if condition is True, the decoding function, converts the message to the corresponding data object. for more information go to [this code snippet](https://github.com/msgpack/msgpack-python#packingunpacking-of-custom-data-type).
- early versions of package does not distinguish between string and binary types and use raw for both of them. but for example in newer versions if you unpack a string with `u''` specifier, it understands that the output object must be string(I think).
- there are something about [extended types](https://github.com/msgpack/msgpack-python#extended-types) which I don't know what they are. becase I have no more time, I will search about them If I need them in funnywing project and in future ...



