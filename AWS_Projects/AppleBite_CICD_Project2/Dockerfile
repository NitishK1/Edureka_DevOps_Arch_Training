# Use the specified base image
FROM devopsedu/webapp

# Set maintainer label
LABEL maintainer="applebite@example.com"
LABEL description="AppleBite PHP Application"

# Set working directory
WORKDIR /var/www/html

# Copy application files to the container
COPY app/ /var/www/html/

# Set permissions
RUN chown -R www-data:www-data /var/www/html && \
    chmod -R 755 /var/www/html

# Expose port 80
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD curl -f http://localhost/ || exit 1

# Start Apache in foreground
CMD ["apache2-foreground"]
