#!/usr/bin/env python
import sys

to_rank = set(["system%d" % i for i in xrange(1,21)])
ranked_list = [line.split()[0] for line in  sys.stdin]
ranked = set(ranked_list)

if len(ranked) != len(ranked_list):
  print "ERROR: your ranked list may not contain duplicates"
for system in to_rank - ranked:
  print "ERROR: %s is missing from your ranked list" % system
for system in ranked - to_rank:
  print "ERROR: your ranked list contains unknown system %s" % system
if len(ranked) != len(ranked_list) or ranked != to_rank:
  sys.exit(1)
