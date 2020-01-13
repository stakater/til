# Socket Hang up error

Socket hang up error occurs when reply from the master exceeds `max_header_length`.

## Solution

Increase `http.max_header_size` value in elasticsearch.yml.

More Details: [here]https://discuss.elastic.co/t/socket-hang-up/87763/4