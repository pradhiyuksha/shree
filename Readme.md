def search(
    self, query: str, 
    num_documents: Optional[int] = None,
    filters: Optional[Dict[str, Any]] = None,
) -> List[Document]:
    """Returns relevant documents matching a query

    Args:
        query: The search query
        num_documents: Number of documents to return. If None, uses self.num_documents
        filters: Optional filters to apply to the search. If None, uses self._default_filters

    Returns:
        List[Document]: List of matching documents
    """
    try:
        if self.vector_db is None:
            logger.warning("No vector db provided")
            return []

        _num_documents = num_documents or self.num_documents
        _filters = filters if filters is not None else getattr(self, '_default_filters', None)

        # Validate and preserve only valid filters, in desired order
        ordered_keys = ['tag', 'company_id', 'company_folder_id']
        temp_filters = {}

        for key in ordered_keys:
            if _filters and key in _filters:
                try:
                    temp_filters[key] = _filters[key]
                except (ValueError, TypeError):
                    logger.warning(f"Invalid {key}: {_filters[key]}")
                    return []

        _filters = temp_filters  # Apply ordered and validated filters

        logger.debug(f"Using filters: {_filters}")
        logger.debug(f"Getting {_num_documents} relevant documents for query: {query}")
        return self.vector_db.search(query=query, limit=_num_documents, filters=_filters)

    except Exception as e:
        logger.error(f"Error searching for documents: {e}")
        return []
