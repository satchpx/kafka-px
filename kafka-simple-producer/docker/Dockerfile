FROM confluentinc/cp-kafka:5.0.1

COPY src/entrypoint /opt/bin/entrypoint
RUN chmod +x /opt/bin/entrypoint

CMD ["/opt/bin/entrypoint"]