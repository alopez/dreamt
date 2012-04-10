#!/usr/bin/env python
import optparse
import sys
import math

optparser = optparse.OptionParser()
optparser.add_option("-k", "--key", dest="key", default="data/answer_key.dev", help="Answer key (default=data/answer_key)")
optparser.add_option("-v", "--verbose", dest="verbose", default=False, action="store_true", help="verbose output (default=false)")
(opts, _) = optparser.parse_args()

human_rank = dict([(line.split()[0],i) for i,line in enumerate(open(opts.key))])
auto_rank = dict([(line.split()[0],i) for i,line in enumerate(sys.stdin)])

if human_rank.keys() != auto_rank.keys():
  sys.stderr.write("ERROR: systems ranked in the input don't match systems ranked in answer key\n")
  sys.exit(1)

if opts.verbose:
  sys.stderr.write("  ")
  for _ in auto_rank:
    sys.stderr.write("   ")
  sys.stderr.write("Human Ranking\n  ")
  for _ in auto_rank:
    sys.stderr.write("---")
  sys.stderr.write("\n")
  for (sysname, rank) in sorted(human_rank.iteritems(), key=lambda x: x[1]):
    sys.stderr.write(" |")
    for (j, _) in enumerate(auto_rank):
      sys.stderr.write(" * " if j==auto_rank[sysname] else "   ")
    sys.stderr.write("| %2d. %s\n" % (rank+1, sysname))
  sys.stderr.write("  ")
  for _ in auto_rank:
    sys.stderr.write("---")
  sys.stderr.write("\n")
  labels = ["%2d. %s" % (rank+1, sysname) for sysname, rank in sorted(auto_rank.iteritems(), key=lambda x: x[1])]
  for k in xrange(max(map(len, labels))):
    sys.stderr.write("  ")
    for label in labels:
      sys.stderr.write(" %s " % label[k] if len(label) > k else "   ")
    sys.stderr.write("\n")
  sys.stderr.write("\n\n")

n = len(auto_rank)  
print 1 - (6*sum([math.pow(human_rank[system]-auto_rank[system],2.0) for system in human_rank]))/(n * (n*n-1))
