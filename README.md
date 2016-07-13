#!/bin/sh
           fermi=$(gawk '/E-fermi/ {print $3}' OUTCAR)                      # rip fermi energy from OUTCAR
           nbands=$(gawk '/NBANDS/ {print $15}' OUTCAR)                      # rip nbands from OUTCAR
           kpts=$(gawk 'NR==2 {print $1;exit}' IBZKPT)                      # rip # of k points from IBZKPT
           nbands2=`echo $nbands+2 | bc`                                    # set line to skip    
           gawk 'NR > 8 {print $2}' EIGENVAL > band_gaptempfile1            # skip first 8 lines
           gawk -v s=$nbands2 'NR%s != 0 {print $1}' band_gaptempfile1 > band_gaptempfile2   # skip k points
           for i in `cat band_gaptempfile2` ; do                            # subtract 
           j=`echo $i - $fermi | bc`                                        #    fermi
           echo "$j" >> band_gaptempfile3                                   #    energy
           done                                                             #    from EIGENVALS
           gawk '/<r>/ min==""|| $1 > 0 {print $1}' band_gaptempfile3 > band_gaptempfile4    # rip + eigenvals
           top=$(gawk 'min==""||min>$1{min=$1}END{print min}' band_gaptempfile4)             # cond band min
           gawk '/<r>/ max==""|| $1 < 0 {print $1}' band_gaptempfile3 > band_gaptempfile5    # rip - eigenvals
           bottom=$(gawk 'max==""||max<$1{max=$1}END{print max}' band_gaptempfile5)          # val band max
           gap=`echo $top - $bottom | bc`                                   # calculate band gap
rm band_gaptempfile*
echo "Written for VASP versions 5.2.2 and 5.3.5"
echo "Fermi energy = $fermi"
echo "Number of bands = $nbands"
echo "Number of k points = $kpts"
echo "CBM = $top"
echo "VBM = $bottom"
echo "band gap = $gap"

# This script works only for noncollinear calculations (i.e. LSORBIT = .TRUE.)
# Proceed with a band structure calculation and then run this script in the same directory as OUTCAR, IBZKPT, and EIGENVAL files.
# Written by John Petersen johnemilpeterseniii@gmail.com
