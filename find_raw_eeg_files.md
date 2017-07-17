# Find common raw EEG data files recursively and collect stats

```bash
#!/bin/bash
#
# The output file log.txt will have the following format:
# /path/to/file_k : owner : file_size_in_bytes : - : time_of_birth time_of_last_access -permissions

rm log.txt
find $1 -regex '.*.\(bdf\|xdf\|edf\|drf\|rdf\|cnt\|asf\|SMA\)$' -exec stat -c '%n : %U : %s : %w : %x' {} + >>log.txt
echo "See output in log.txt"
