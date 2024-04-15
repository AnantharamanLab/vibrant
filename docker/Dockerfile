FROM ghcr.io/charlenelawdes/hmmer-docker AS hmmer-builder
FROM ghcr.io/charlenelawdes/prodigal-docker AS prodigal-builder
FROM ghcr.io/charlenelawdes/sklearn-docker AS sklearn-builder

FROM debian:bullseye
RUN apt-get update && apt-get install -y \
    git \
    gcc \
    python3 \
    cython3 \
    python3-biopython \
    python3-matplotlib \
    python3-pandas \
    python3-seaborn \
    python3-pip \
    wget

# Copy executables from builder images
COPY --from=hmmer-builder /usr/local/bin/ /usr/local/bin/
COPY --from=prodigal-builder /usr/local/bin/ /usr/local/bin/

# Install scikit-learn for build stage with the version required by VIBRANT
RUN pip3 install -U scikit-learn==0.21.3

# Install VIBRANT
RUN git clone https://github.com/AnantharamanLab/VIBRANT
RUN chmod -R 777 VIBRANT

# Setup the databases files 
WORKDIR /VIBRANT/databases


RUN wget http://fileshare.csb.univie.ac.at/vog/vog94/vog.hmm.tar.gz && \
    wget ftp://ftp.ebi.ac.uk/pub/databases/Pfam/releases/Pfam32.0/Pfam-A.hmm.gz && \
    wget ftp://ftp.genome.jp/pub/db/kofam/archives/2019-08-10/profiles.tar.gz


RUN tar -xzf vog.hmm.tar.gz && \
    gunzip Pfam-A.hmm.gz && \
    tar -xzf profiles.tar.gz


RUN cat VOG*.hmm > vog_temp.HMM && \
    cat profiles/K*.hmm > kegg_temp.HMM


RUN rm VOG0*.hmm VOG1*.hmm VOG2*.hmm && \
    rm -R profiles


RUN hmmfetch -o VOGDB94_phage.HMM -f vog_temp.HMM profile_names/VIBRANT_vog_profiles.txt && \
    hmmfetch -o KEGG_profiles_prokaryotes.HMM -f kegg_temp.HMM profile_names/VIBRANT_kegg_profiles.txt && \
    mv Pfam-A.hmm Pfam-A_v32.HMM


RUN rm vog_temp.HMM kegg_temp.HMM vog.hmm.tar.gz profiles.tar.gz


RUN hmmpress VOGDB94_phage.HMM && \
    hmmpress KEGG_profiles_prokaryotes.HMM && \
    hmmpress Pfam-A_v32.HMM

# Verify databases and dependencies
RUN python3 VIBRANT_setup.py -test

# Set the working directory back to /VIBRANT
WORKDIR /VIBRANT
